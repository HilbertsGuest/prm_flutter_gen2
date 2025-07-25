# PRM_Brainstorm.md
**Working title:** PRM System (Personal Relations Management)  
**Date:** 2025‑07‑25

---

## 1. Problem & Value Proposition

**Problem:** People forget key personal details and prior topics, weakening relationships or making interactions awkward (“I know this person from that party… but what’s their name again?”).

**Job-To-Be-Done (JTBD):**  
> “Help me recall the right details and topics so I can make others feel seen and valued—without relying on my memory.”

**Core Value Prop (draft):**  
A lightweight, personal CRM that lets you quickly capture post-conversation notes and instantly retrieve them by person, event, topic, or fuzzy crumbs—so you always show up prepared and thoughtful.

---

## 2. Primary Users & Context

- **Users:** Individuals who actively maintain broad or meaningful networks (students, founders, freelancers, community organizers).  
- **Context of Use:** Right after interactions (capture mode) and right before/at the next meeting (recall mode).  
- **Critical Moment:** When you can’t remember a person’s name/context but have fragments (event, topic) to search.

---

## 3. Constraints & Assumptions

- **Platform:** Flutter (mobile-first; Android/iOS).  
- **Data:** Likely offline-first with local storage; sync optional.  
- **Search:** Avoid Android’s problematic FTS5 integration that caused data loading issues in prior attempts.  
- **Privacy:** Personal data must feel secure (biometric lock, local encryption considered).  
- **Scope:** Focus on individual use (not team CRM) for V1.

---

## 4. Lessons from Previous Attempts

- Heavy reliance on SQLite FTS5 led to loading/sync headaches on Android.  
- Need a simpler, more reliable local search/indexing approach (e.g., Isar/ObjectBox/custom index) to de-risk.

---

## 5. Ideation – Round A (“Obvious Firsts”)

### 5.1 Capture (Post-interaction)
- **Quick Add Sheet:** Minimal form (Name/Alias, event/context, topics, follow-ups).  
- **Voice-to-Text Capture:** Dictate notes; auto-transcribe.  
- **Photo/Card Snap + OCR:** Extract names/keywords from business cards or quick pics.  
- **Auto-templates:** “Met at ___, talked about ___, next time ask about ___.”  
- **One-tap Tags:** Party, work, travel, book rec, health, etc.

### 5.2 Organize / Relationship Model
- **Contact Profiles:** Person page with basics, last interaction, key facts.  
- **Interaction Timeline:** Chronological log per person.  
- **Topic/Interest Library:** Structured info per contact (books, hobbies, allergies).  
- **Event Entities:** Parties/conferences as first-class searchable objects.  
- **Custom Fields:** Let users add bespoke fields (e.g., “kid’s names”, “dietary prefs”).

### 5.3 Recall / Search
- **Full-text Search with Filters:** Search across all notes, filter by event/date/tag/person.  
- **Context Search (“party X”)**: Retrieve all people tied to that event.  
- **Smart Suggestions:** Before meetings, surface last mentioned topics (“Alex → moving to Berlin”).  
- **Name-Rescue Mode:** Quick wizard—type a few crumbs (place, topic) → probable matches.  
- **Fuzzy Matching/Synonyms:** “Berlin move” finds “relocating to Germany”.

### 5.4 Act / Follow-up
- **Follow-up Reminders:** “Ping in 3 weeks about job interview.”  
- **Cadence Tracking:** “Keep in touch every X months.”  
- **Prep Cards:** Auto-generate a brief before a meeting.  
- **Task Integration:** Export follow-ups to task managers or keep a built-in list.  
- **Birthday/Anniversary Import:** Gentle nudges from contacts/calendars.

### 5.5 Delight / Meta
- **Relationship Health Score:** Simple heuristic (recency, depth, reciprocity).  
- **Event Warm-up Prompts:** Social hacks before gatherings.  
- **Mini AI Coach:** Suggest phrasing/icebreakers.  
- **Privacy & Encryption Controls.**  
- **Offline-first Mode** with background sync when online.

---

## 6. Candidate V1 Feature Shortlist (TO CONFIRM)

*(Draft pick—mark KEEP/CUT/TBD)*

**Capture:**  
- Quick Add Sheet (KEEP)  
- One-tap Tags (KEEP)  
- Voice-to-Text (TBD)  
- OCR Card Snap (CUT or v2?)

**Organize:**  
- Contact Profiles (KEEP)  
- Interaction Timeline (KEEP)  
- Event Entities (KEEP)  
- Custom Fields (TBD)

**Search/Recall:**  
- Full-text Search + Filters (KEEP; but choose safe tech)  
- Context/Event Search (KEEP)  
- Name-Rescue Mode (TBD)  
- Smart Suggestions (v2 if AI needed)

**Follow-up:**  
- Follow-up Reminders (KEEP)  
- Prep Cards (TBD)  
- Cadence Tracking (TBD)  
- Task Integration (CUT or v2)

**Delight/Meta:**  
- Privacy Lock (KEEP)  
- Relationship Health Score (CUT/v2)  
- Mini AI Coach (CUT/v2)

---

## 7. Open Questions / Risks

- **Local Search Engine Choice:** Which DB/index library best balances speed, reliability, and dev ease?  
- **Sync/Backup:** Local-only at first? Optional cloud backup? How?  
- **AI Features:** Manual rules first or integrate LLM later?  
- **UX Friction:** How to make capture <30 seconds? (Templates, voice, quick actions).  
- **Data Sensitivity:** How to reassure users their notes are private? (Encryption, no server default).

---

## 8. Next Steps

1. **Confirm V1 shortlist** (mark each item Keep/Cut/TBD).  
2. **Run Round B (Twist/SCAMPER) for creative low-cost wins**.  
3. **Cluster & score ideas (ICE/RICE) to finalize scope.**  
4. **Draft PRD** (problems, users, success metrics, feature list).  
5. **Architecture doc** (data model, search layer decision, offline strategy).  
6. **De-risk spike**: prototype the chosen local DB/search solution.

---

## 9. Commands / Workflow Hooks

- `*brainstorm roundB` – Creative twist round.  
- `*cluster` – Group & theme ideas.  
- `*score` – Prioritize with ICE/RICE.  
- `*v1 shortlist` – Finalize the must-build set.  
- `*pm create-doc prd` – Spin up the PRD.  
- `*architect create-doc architecture` – Start technical design.  
- `*end-brainstorm` – Wrap & handoff.

---