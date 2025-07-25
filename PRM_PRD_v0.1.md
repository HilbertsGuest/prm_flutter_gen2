# Product Requirements Document (PRD)
**Product:** PRM System (Personal Relations Management)  
**Version:** v0.1 (Draft)  
**Date:** 2025-07-25  
**Owner:** PM (you) / Assistant PM (ChatGPT)

---

## 1. Overview

A personal relationship manager that lets individuals quickly capture post‑interaction notes and reliably recall them later by person, event, topic, or fuzzy clues—so they can make others feel seen and valued without memorizing everything.

---

## 2. Problem Statement

People forget names, contexts, and personal details from past conversations. This creates awkward moments (“I know you from that party… but your name?”) and weakens relationship quality. Existing CRMs are heavy/business-centric; notes apps are unstructured and hard to search contextually.

---

## 3. Goals & Success Metrics (DRAFT – needs numbers)

**Primary Goals**
- Rapid capture (<30 seconds) of post‑interaction notes.
- Near-instant recall (<10 seconds) of the right person/details using minimal crumbs (event, topic).

**Success Metrics (initial proposals)**
- **Activation:** ≥60% of new users add 3+ contacts and 1 follow-up within first 7 days.  
- **Recall Success Rate:** ≥80% of searches end with the user confirming they found what they needed within 10 seconds.  
- **Retention (30-day):** ≥35% of users return and log at least one new interaction in month 1.  
- **Median Time-to-Capture:** <25 seconds (instrumented).  
- *(TBD: qualitative metric like “felt more prepared” survey result ≥70% positive)*

---

## 4. Target Users & Personas

**Primary Persona – “The Connector Individual”**  
Students, founders, freelancers, community organizers—anyone who meets many people and wants to nurture those relationships intentionally.

**Context of Use**  
- **Capture Mode:** Immediately after a conversation/meeting.  
- **Recall Mode:** Just before or during the next interaction.

---

## 5. Scope (MoSCoW)

### 5.1 Must-Have (V1)
| Area     | Feature                                                | Notes                                           |
|----------|--------------------------------------------------------|-------------------------------------------------|
| Capture  | Quick Add Sheet (name/alias, event/context, topics, follow-ups) | Minimal friction, <30s target                  |
| Capture  | One-tap Tags                                           | Predefined & custom tags                        |
| Organize | Contact Profiles & Interaction Timeline                | Person-centric history                          |
| Organize | Event/Context entities                                 | Parties, conferences as first-class objects     |
| Search   | Full-text + filters (person/event/tag/date)            | Reliable local index (avoid FTS5 issues)        |
| Follow-up| Simple reminders for follow-ups                        | Local notifications fine for V1                 |
| Privacy  | App lock (PIN/biometric)                               | Optional encryption-at-rest (if feasible)       |

### 5.2 Should-Have (if capacity)
- Name-Rescue Mode (crumb-based wizard)  
- Custom Fields on contacts  
- Prep Cards (auto-brief before meeting)  
- Voice-to-text capture

### 5.3 Could-Have (vNext / Nice-to-haves)
- OCR card/photo ingestion  
- Cadence tracking (“touch every X months”)  
- Birthday/anniversary import

### 5.4 Won’t-Have (explicitly out for V1)
- Relationship Health Score  
- AI coach for phrasing  
- Task manager integrations  
- Team/shared workspaces

---

## 6. Non-Functional Requirements

- **Platform:** Flutter (Android & iOS).  
- **Offline-first:** All core flows work without network.  
- **Local Data Store:** Reliable, fast, easy to bundle (e.g., Isar, ObjectBox, Drift).  
- **Performance:** Search results in <300 ms for typical datasets (<5k notes).  
- **Privacy & Security:** Local-only by default. Optional encrypted backup (TBD).

---

## 7. Dependencies & Technical Considerations

- **Local search/indexing engine choice** (must avoid prior FTS5 pain).  
- **Notification APIs** (iOS/Android) for follow-ups.  
- **Optional cloud backup solution** (later: iCloud/Google Drive or custom).

---

## 8. Risks & Mitigations

| Risk                               | Impact        | Mitigation                                              |
|------------------------------------|---------------|---------------------------------------------------------|
| DB/search tech fails again         | App unusable  | Spike prototype; choose proven Flutter-friendly DB      |
| Scope creep (too many “nice” features) | Delay MVP  | Strict MoSCoW; lock V1 after scoring                    |
| UX friction in capture             | Low adoption  | Template-driven capture; measure capture time           |
| Privacy concerns                   | User churn    | Local-first, clear privacy messaging, biometric lock    |

---

## 9. Release Strategy

**MVP (Beta) Milestones:**  
1. **Spike:** Evaluate 2–3 local DB/search options (1–2 weeks).  
2. **MVP Build:** Must-have features only (4–6 weeks).  
3. **Closed Beta:** Test with 10–20 users, capture telemetry & feedback (2 weeks).  
4. **V1 Launch:** Store release + basic onboarding/tutorial.

---

## 10. Open Questions

- What exact quantitative targets feel right for success metrics?  
- Do we want any sync/backup at launch, or purely local?  
- Which “Should” features are truly critical for delight vs. can slip?  
- Any legal/compliance requirements (GDPR if cloud sync later)?  
- Do we need desktop/web companion later?

---

## 11. Next Steps & Commands

- Confirm V1 shortlist (KEEP/CUT/TBD).  
- Run creative Round B (SCAMPER) if needed: `*brainstorm roundB`  
- Cluster & score ideas: `*cluster`, `*score`  
- Finalize PRD: `*pm revise <section>`  
- Hand off to Architect: `*architect create-doc architecture`  
- Export updates anytime: `*pm export prd`
