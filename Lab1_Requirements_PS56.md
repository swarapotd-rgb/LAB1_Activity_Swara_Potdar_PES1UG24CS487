# Lab 1 — Requirements Engineering

**Problem Statement #56 — Peer Skill Exchange & Mentorship Network**
Domain: Media, Events & Community

---

## 1. Scenario Summary

A time-banking skill exchange platform where community members trade teaching services using **time credits** instead of money (e.g. 1 hour of Python tutoring ↔ 1 hour of Guitar lessons). A learner locates a mentor, books and attends a session, both parties verify completion and rate each other, and the mentor's time-credit balance is credited. The platform's schedule must integrate with external calendar services.

**Given by the problem statement:** FR-001 (time-credit balance), NFR-001 (calendar sync), actors *Learner Member* and *Skill Mentor*, mutual feedback rating validation.

---

## 2. Requirements Table

### 2.1 Functional Requirements

| Req ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|---|
| **FR-001** | Functional | The system shall maintain a time-credit balance for each member, crediting 1 hour upon verified completion of a teaching session. | High | **Pass:** 1 credit is added to the mentor's balance and 1 deducted from the learner's within 1 minute of a session being marked verified. **Fail:** Balances are unchanged, or changed before verification. | Time credit is the platform's currency; the entire exchange model depends on it. *(Given)* |
| **FR-002** | Functional | The system shall allow a member to register, edit, and remove the skills they are available to teach, each with a skill name and proficiency level. | High | **Pass:** A registered skill appears on the member's mentor profile and is returned by a learner search for that skill. **Fail:** A registered skill does not appear on the profile or in search results. | Without an advertised skill inventory, no learner can locate a mentor. |
| **FR-003** | Functional | The system shall allow a learner to search registered mentors by skill name and display only mentors who have at least one available time slot. | High | **Pass:** Searching "Python" returns every mentor offering Python with ≥1 open slot, and no mentor without one. **Fail:** A qualifying mentor is omitted, or a mentor with no availability is returned. | Matching learners to mentors is the core discovery function of the exchange. |
| **FR-004** | Functional | The system shall allow a learner holding a time-credit balance of at least 1 hour to book an available slot with a selected mentor, and shall reject the booking otherwise. | High | **Pass:** With a balance ≥ 1, the booking is created and the slot is marked unavailable; with a balance of 0, the booking is rejected with a message stating credits are required. **Fail:** A booking is created for a learner with 0 credits, or an available slot is not reserved after a valid booking. | Enforces the fail condition stated in FR-001 and creates the scheduled session. |
| **FR-005** | Functional | The system shall require both the learner and the mentor to confirm completion of a scheduled session and submit a mutual rating (1–5) before the session is marked verified. | High | **Pass:** The session is marked verified only after both confirmations and both ratings are recorded. **Fail:** The session is marked verified with one or zero confirmations, or with a missing rating. | Prevents fraudulent credit claims and delivers the mutual feedback rating validation named in the problem statement. |

### 2.2 Non-Functional Requirements

| Req ID | Type | Description | Priority | Acceptance Criteria | Rationale |
|---|---|---|---|---|---|
| **NFR-001** | Nonfunctional | The session scheduling calendar shall sync seamlessly with Google Calendar and Outlook via iCal export feeds. | High | **Pass:** A newly booked session appears in a subscribed Google Calendar and Outlook feed with the correct date, time, and duration within 5 minutes. **Fail:** The session is absent after 5 minutes or shows an incorrect time. | Members manage their time in existing calendars; unsynced sessions cause no-shows. *(Given)* |
| **NFR-002** | Nonfunctional | The system shall reject 100% of attempts to alter a member's time-credit balance that do not originate from a verified session transaction. | High | **Pass:** All direct modification attempts by any account, including the balance owner, are rejected and logged. **Fail:** Any single attempt succeeds in changing a balance outside a verified session. | Time credits carry real value; a forgeable balance collapses trust in the whole exchange. |

---

## 3. Peer Critique & Revision Log

| Req ID | Issue Identified | Revision Made |
|---|---|---|
| FR-002 | "List skills" was ambiguous — display vs. register. | Reworded to "register, edit, and remove"; added proficiency level. |
| FR-003 | Search returned mentors regardless of availability, making results unactionable. | Restricted results to mentors with ≥1 available slot. |
| FR-004 | The credit check implied by FR-001's fail condition had no requirement enforcing it. | Added the ≥1 credit precondition and explicit rejection behaviour. |
| FR-005 | "Verified by participating members" did not say how many confirmations. | Specified mutual confirmation by both parties; merged in the mandatory 1–5 rating so the problem statement's feedback requirement is covered within the 5-FR limit. |
| NFR-001 | No acceptance criterion supplied with the given requirement. | Added a measurable 5-minute sync window with correct date/time/duration. |
| NFR-002 | Original wording ("prevent unauthorized users") read as functional access control and was not measurable. | Reframed as a 100%-rejection integrity constraint scoped to non-transactional modification. |

---

## 4. Actors & Use Cases

### 4.1 Actors

| # | Actor | Type | Purpose |
|---|---|---|---|
| A1 | **Learner Member** | Person (primary) | Searches for skills, books sessions, spends time credits, confirms completion and rates the mentor. |
| A2 | **Skill Mentor** | Person (primary) | Registers teachable skills, publishes availability, conducts sessions, confirms completion, earns time credits. |
| A3 | **Calendar Service** | External system (secondary) | Receives iCal feeds of booked sessions for Google Calendar / Outlook. Justified by NFR-001. |

> A single person may hold both the Learner Member and Skill Mentor roles; they are modelled separately because each role interacts with the system for a different purpose.

### 4.2 Use Cases

| # | Use Case | Primary Actor | Traces to |
|---|---|---|---|
| UC1 | Register Teaching Skill | Skill Mentor | FR-002 |
| UC2 | Manage Availability | Skill Mentor | FR-003, FR-004 |
| UC3 | Search for Mentor | Learner Member | FR-003 |
| UC4 | Book Skill Session | Learner Member | FR-004 |
| UC5 | Verify Session Completion | Learner Member, Skill Mentor | FR-005 |
| UC6 | Manage Time-Credit Balance | *(system-internal, triggered)* | FR-001, NFR-002 |
| UC7 | Check Time-Credit Balance | — (included) | FR-004 |
| UC8 | Sync Session to External Calendar | Calendar Service | NFR-001 |
| UC9 | Add Written Feedback Comment | Learner Member, Skill Mentor | FR-005 (optional part) |

### 4.3 Relationships

**`<<include>>` — mandatory behaviour**

- `Book Skill Session` **includes** `Check Time-Credit Balance`
  Every booking must verify the learner holds ≥1 credit before the slot is reserved (FR-004).
- `Verify Session Completion` **includes** `Manage Time-Credit Balance`
  Every verification triggers the credit transfer (FR-001).

**`<<extend>>` — optional / conditional behaviour**

- `Add Written Feedback Comment` **extends** `Verify Session Completion`
  The numeric rating is mandatory, but an accompanying written comment is optional.
- `Sync Session to External Calendar` **extends** `Book Skill Session`
  Only occurs if the member has connected a Google Calendar or Outlook feed (NFR-001).

### 4.4 Diagram Layout (for Step 5)

```text
                 ┌──────────────── Peer Skill Exchange & Mentorship Network ─────────────────┐
                 │                                                                           │
                 │   (Register Teaching Skill)          (Manage Time-Credit Balance)         │
  Skill Mentor ──┼──                                              ▲                          │
       ○         │   (Manage Availability)                        │ <<include>>              │
      /|\        │                                                │                          │
      / \        │   (Verify Session Completion) ─────────────────┘                          │
                 │            ▲                                                              │
                 │            │ <<extend>>                                                   │
                 │   (Add Written Feedback Comment)                                          │
                 │                                                                           │
  Learner       ─┼──  (Search for Mentor)                                                    │
  Member         │                                                                           │
       ○         │    (Book Skill Session) ── <<include>> ──▶ (Check Time-Credit Balance)    │
      /|\        │            ▲                                                              │
      / \        │            │ <<extend>>                                                   │
                 │   (Sync Session to External Calendar) ────────────────────────────────────┼── ○ Calendar
                 │                                                                           │     Service
                 └───────────────────────────────────────────────────────────────────────────┘
```

Actors sit outside the system boundary rectangle; use cases are ovals inside it. Solid lines = associations, dashed arrows with stereotype labels = include/extend.

---

## 5. Deliverable Checklist

- [x] Step 1 — Scenario review
- [x] Step 2A — Brainstorm functions & constraints
- [x] Step 2B — Requirements table (5 FR + 2 NFR)
- [x] Step 3 — Peer critique & revision
- [x] Step 4 — Actors (3) & use cases (9)
- [ ] Step 5 — UML use-case diagram in draw.io → export PDF
- [ ] Step 6 — Use-case flow for *Book Skill Session* → export PDF
- [ ] Step 7 — Push both PDFs to Lab-1 Git repository
