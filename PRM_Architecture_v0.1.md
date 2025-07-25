# Architecture Doc – PRM System (v0.1 Draft)
*Role: Architect* • *Date: 2025-07-25* • *Platform: Flutter (Android/iOS)*

---

## 0. Purpose & Scope

Define a **thin, stable architecture** for V1 of the PRM app: capture → organize → recall → follow‑up. Emphasis on **offline-first**, **fast local search**, and avoiding prior FTS5 pain. This doc will guide implementation, de‑risk key tech choices, and serve as the baseline for ADRs (Architecture Decision Records).

---

## 1. High-Level Architecture

```
┌──────────────────────────────────────────┐
│              Presentation                │
│  Flutter UI (Feature modules):           │
│   - Capture Flow  - Search/Recall Flow   │
│   - Contacts/Profile  - Follow-up/Remind │
│   State mgmt: e.g., Riverpod/Bloc        │
└───────────────▲──────────────────────────┘
                │ UseCases (async, pure)
┌───────────────┴──────────────────────────┐
│                Domain                    │
│  Entities (Contact, Interaction, Event…) │
│  Value Objects, Repositories (abstract)  │
│  Use Cases / Services (AddNote, Search…) │
└───────────────▲──────────────────────────┘
                │ Impl of repos/adapters
┌───────────────┴──────────────────────────┐
│                 Data                     │
│  Local DB (Isar/ObjectBox/Drift?)        │
│  Local Search Index (custom/in-DB)       │
│  Notification Adapter (iOS/Android)      │
│  Optional OCR/Voice services (plugins)   │
│  Optional Cloud Backup adapter (later)   │
└──────────────────────────────────────────┘
```

- **Clean layering:** UI ↔ Domain via Use Cases; Data layer hidden behind repositories.  
- **Dependency Rule:** Inner layers know nothing about outer ones.  
- **DI Container:** (`get_it` or `riverpod` providers) to wire implementations.

---

## 2. Key Decisions (Draft ADRs)

| ID | Decision Topic | Choice (Draft) | Status |
|----|----------------|----------------|--------|
| ADR-001 | Local DB & Search | **Isar** for object DB + custom lightweight inverted index for full-text search (or Isar `.contains()` fallback) | Pending spike |
| ADR-002 | State Mgmt | **Riverpod** (simple, testable, compile-time safety) | Tentative |
| ADR-003 | Offline-first Strategy | Local-first, background sync none in V1 (manual export/backup TBD) | Accepted |
| ADR-004 | Notifications | Native local notifications via `flutter_local_notifications` | Accepted |
| ADR-005 | Encryption/Lock | App-level lock (PIN/biometric); DB-level encryption optional (Isar has none → wrap with OS keystore or encrypted file system) | Needs research |
| ADR-006 | Search Tokenizer | Simple whitespace + punctuation split, lowercase, stopword removal. Persist token→noteIds map. | Pending spike |
| ADR-007 | OCR/Voice | Exclude from V1 (keep plugin stubs). | Accepted |

> We’ll write each ADR in its own file once confirmed.

**Why not FTS5 again?** Previous Android loading issues. We’ll avoid shipping custom sqlite unless spike proves trivial. If Drift+sqlite3 with bundled `sqlite3_flutter_libs` + FTS5 is painless, ADR-001 could switch. That’s why we spike both.

---

## 3. Core Modules & Responsibilities

### 3.1 Feature Modules (Presentation Layer)

- **Capture Module**  
  - Screens: Quick Add, Voice Note (TBD), Tag Picker  
  - ViewModels/Controllers: `CaptureController` (validates, calls `AddInteractionUseCase`)

- **Search/Recall Module**  
  - Screens: Global Search, Filter Panel, Name-Rescue wizard  
  - Controller: `SearchController` → `SearchUseCase`

- **Contacts/Profile Module**  
  - Screens: Contact List, Contact Detail (timeline view), Event Detail  
  - Controller: `ContactController`, `EventController`

- **Follow-up/Reminder Module**  
  - Screens: Reminder List, Schedule Reminder  
  - Controller: `ReminderController`

- **Settings/Privacy Module**  
  - Screens: App Lock settings, Export/Import (future)  
  - Controller: `SettingsController`

### 3.2 Domain Layer

**Entities (immutable, pure Dart):**

```dart
class Contact {
  final ContactId id;
  final String displayName;
  final List<String> tags;
  final DateTime? birthday;
  // ...
}

class Interaction {
  final InteractionId id;
  final ContactId contactId;
  final EventId? eventId;
  final DateTime timestamp;
  final String note;            // raw text
  final List<String> topics;    // extracted/selected
  final List<FollowUp> followUps;
}

class Event {
  final EventId id;
  final String name;
  final DateTime? date;
  final String? location;
  final List<ContactId> participants; // or derive from interactions
}

class FollowUp {
  final String description;
  final DateTime dueAt;
  final bool done;
}
```

**Repositories (abstract):**

```dart
abstract class ContactRepository {
  Future<Contact> create(Contact c);
  Future<Contact?> get(ContactId id);
  Stream<List<Contact>> watchAll();
  Future<void> update(Contact c);
  Future<void> delete(ContactId id);
}

abstract class InteractionRepository { /* CRUD, query by contact/event/date */ }
abstract class SearchRepository {
  Future<List<SearchResult>> search(String query, {Filters? filters});
}
abstract class ReminderRepository { /* CRUD + scheduling */ }
```

**Use Cases (application services):**
- `AddInteractionUseCase`
- `SearchNotesUseCase`
- `CreateReminderUseCase`
- `GetPrepCardUseCase` (TBD)
- `ListContactsUseCase`, `GetContactProfileUseCase`, etc.

### 3.3 Data Layer

**Option A (Preferred initial): `Isar`**  
- Pros: Fast, async, easy to use in Flutter, no native SQL headaches.  
- Cons: No true FTS; need custom index or rely on `.contains()`, which may be slower.  
- Mitigation: Build an inverted index collection `SearchToken { token, Set<InteractionId> ids }`.

**Option B: Drift + sqlite3 + FTS5 (bundled)**  
- Pros: Real full-text search, mature SQL, declarative schema.  
- Cons: Must ship native sqlite libs; more setup/edge cases; previous pain risk.

**Custom Search Index Flow (if Isar):**

1. On save/update of `Interaction.note`, tokenize.  
2. For each token, upsert `SearchToken(token).ids.add(interactionId)`  
3. Search: tokenize query tokens, intersect sets of ids, then sort by recency/relevance.

**Notifications:**  
- `flutter_local_notifications` plugin. Repos abstract scheduling.

**Encryption & Lock:**  
- App-level lock: biometrics + passcode; store a key in Keychain/Keystore if DB encryption adopted (e.g., wrap raw file).  
- If encryption deferred, at least guard UI with lock.

**Backup/Export (post-V1):**  
- JSON export to file system. Later: cloud drive integration.

---

## 4. Data Model (ER Sketch)

```
Contact (id PK)
  - displayName
  - tags [String]
  - ...

Interaction (id PK)
  - contactId FK -> Contact.id
  - eventId FK -> Event.id (nullable)
  - timestamp (DateTime)
  - note (Text)
  - topics [String]
  - followUps [Embedded]

Event (id PK)
  - name
  - date
  - location
  - ...

SearchToken (token PK)
  - token (String)
  - interactionIds [List<InteractionId>]  // or separate entry table
```

---

## 5. Flows & Sequence Diagrams (Textual)

### 5.1 Capture Flow

```
User -> UI: Open Quick Add
UI -> CaptureController: submit form
CaptureController -> AddInteractionUseCase: execute(data)
AddInteractionUseCase -> InteractionRepo: save(interaction)
AddInteractionUseCase -> SearchRepo: updateIndex(tokens, interactionId)
InteractionRepo -> AddInteractionUseCase: OK
AddInteractionUseCase -> UI: success
UI -> Notifications: schedule follow-up (optional)
```

### 5.2 Search/Recall Flow

```
User -> UI: enters query "party x"
UI -> SearchController: search(query, filters)
SearchController -> SearchUseCase: execute(query)
SearchUseCase -> SearchRepo: searchTokens(queryTokens)
SearchRepo -> SearchUseCase: interactionIds[]
SearchUseCase -> InteractionRepo: fetch details
SearchUseCase -> UI: results sorted
UI: show candidates; user picks contact
```

---

## 6. Non-Functional Requirements Mapped

| NFR | Implementation Hook |
|-----|---------------------|
| Offline-first | All repos read/write local DB; no network dependencies |
| Performance | Usecases return in <300ms. Indexing async. Debounce search input. |
| Security | App lock screen; optional encrypted storage. |
| Reliability | Indexed operations tested; migration scripts for DB schema versions. |
| Testability | Use cases pure; repos mocked with in-memory impl for unit tests. |

---

## 7. Testing Strategy

- **Unit Tests:**  
  - Use cases (AddInteraction, Search).  
  - Tokenizer/index builder.  
- **Widget Tests:**  
  - Capture form validation.  
  - Search UI.  
- **Integration Tests:**  
  - DB migrations.  
  - Notification scheduling.  
- **Performance Tests:**  
  - Search speed with 5k notes.  
- **Manual Beta Protocol:**  
  - Scenario scripts (capture after event, recall before meeting).

---

## 8. Build & Release Pipeline

- **CI (GitHub Actions / GitLab CI):**  
  - Lint, tests, format check.  
  - Build Android/iOS artifacts for beta (Firebase App Distribution / TestFlight).  
- **Versioning:**  
  - Semantic for code; in-app DB schema versioning with migration scripts.  
- **Feature Flags:**  
  - Gate experimental features (e.g., OCR) after MVP.

---

## 9. Open Questions

1. **Definitive DB/Search choice** after spike (1–2 days).  
2. **Encryption needs**: Is app lock enough, or full DB encryption required?  
3. **Prep Cards (auto-brief)**: Use simple templates or integrate AI later?  
4. **Name-Rescue Mode** complexity: is fuzzy match essential V1?  
5. **Export/Backup** approach for early adopters (manual vs cloud).  
6. **Telemetry/Analytics**: Local-only metrics vs anonymous remote analytics.

---

## 10. Next Steps

1. **Run Tech Spike (ADR-001):** Compare Isar vs Drift+FTS5 for:  
   - Insert/search speed, complexity, binary size, platform quirks.  
2. **Lock ADRs** & update this doc to v1.0.  
3. **Generate Data Schemas & Repo interfaces**.  
4. **Scaffold project structure (packages/directories)**.  
5. **Kick off implementation with Capture & Search first.**

---

### Commands / Hand-offs

- `*architect revise <section>` – Update part of this doc  
- `*architect adr <topic>` – Create a specific ADR  
- `*dev start-skeleton` – Hand off to Dev for project scaffold  
- `*pm sync` – Ensure PRD & Arch stay aligned
