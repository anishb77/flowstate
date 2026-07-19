# FLOWSTATE Engineering Implementation Roadmap (Revised)

## Milestone 1: Core Data Models & Database Foundation

**Objective:** Establish the primary database schema, explicitly separating parent templates from spawned daily instances, and configure the write-only event log.

- **Relevant PDS Chapters:** 7.1 (Templates vs. Clones), 7.2 (Data Isolation), 7.3 (Text-Array Storage), 7.4 (Event Log).
- **Dependencies:** None.
- **Deliverables:**
- `parent_templates` and `daily_task_instances` table schemas.
- `interaction_event_log` table schema (append/write-only).
- ORM/Query builder configuration supporting JSON/text-array serialization.

- **Acceptance Criteria:**
- Updates to a spawned daily instance do not mutate the parent template.
- Database schema is successfully created and accessible by the application.
- CRUD operations on the core entities succeed.

- **Suggested Git Branch:** `feat/m1-data-foundation`

## Milestone 2: Authentication & User Foundation

**Objective:** Establish secure user access, identity management, and protected routing so all subsequent data can be tied to authenticated profiles.

- **Relevant PDS Chapters:** Core Architecture (Implicit requirement for multi-tenant data isolation).
- **Dependencies:** Milestone 1.
- **Deliverables:**
- Supabase Auth configuration.
- User profile creation pipeline.
- Login and logout UI flows.
- Frontend route protection and session persistence.

- **Acceptance Criteria:**
- Unauthenticated users are strictly redirected to the login screen.
- User sessions persist across page reloads and application restarts.

- **Suggested Git Branch:** `feat/m2-authentication`

## Milestone 3: Administrative Vault & CRUD UI

**Objective:** Build the quarantined administrative interfaces so developers and QA can input data to test downstream engines.

- **Relevant PDS Chapters:** 2.2 (Core Behavior Rules - Isolation), 10.3 (Step 2/3 - Vault & Forms).
- **Dependencies:** Milestone 1, Milestone 2.
- **Deliverables:**
- Task Creation Form (supporting the 5-second Starting Steps array).
- Creation Vault (List view of active parent templates).
- History Archive (Read-only view of past interactions).

- **Acceptance Criteria:**
- Authenticated users can successfully create, edit, and delete their own parent templates.
- Starting steps are successfully saved and retrieved as arrays.

- **Suggested Git Branch:** `feat/m3-admin-vault`

## Milestone 4: Automated Task Lifecycle Engine

**Objective:** Implement the background cron systems that spawn, expire, and lock tasks based on temporal rules and user behavior.

- **Relevant PDS Chapters:** 8.1 (On-Demand Gen), 8.2 (Auto-Locking), 8.3 (4:00 AM Reset), 8.4 (Stopping Recurring).
- **Dependencies:** Milestone 3.
- **Deliverables:**
- On-launch task generation service.
- Overnight 4:00 AM local cleanup and expiration service.
- Skip-threshold counter and auto-locking state mutation on templates.

- **Acceptance Criteria:**
- `Assert(instances.count == active_templates.count)` on first calendar day launch.
- Unfinished tasks transition to `status == 'expired'` exactly at 4:00 AM local time.
- Skipping an active instance 3 consecutive times securely locks the parent template.

- **Suggested Git Branch:** `feat/m4-lifecycle-engine`

## Milestone 5: State Inference & Local Behavior Engine

**Objective:** Build the telemetry analysis math and the fallback offline pipeline to serve tasks instantly.

- **Relevant PDS Chapters:** 3.1 - 3.3 (State Inference & Archetypes), 5.1 - 5.3 (Local Behavior Engine).
- **Dependencies:** Milestone 4.
- **Deliverables:**
- Momentum score calculator (0-100 scale) and data decay weighting math.
- Stage 1 rule-based pipeline (screens tasks against availability/deadlines).
- Local heuristic ranking algorithm.

- **Acceptance Criteria:**
- Engine returns exactly one top-ranked task suggestion instantly (under 50ms).
- Momentum score accurately drops when skip events are injected into the log.

- **Suggested Git Branch:** `feat/m5-local-engine`

## Milestone 6: Core Action Hub (Focus UI) & Idle Management

**Objective:** Build the single-action daily dashboard and integrate the UI-driven idle detection listeners.

- **Relevant PDS Chapters:** 9.1 - 9.7 (Interface & UX System), 13.1 (Handling Idle Time).
- **Dependencies:** Milestone 5.
- **Deliverables:**
- Single-task Action Hub view (high-contrast inversion, 5-second step display).
- System-level input monitor (mouse, keyboard, focus state).
- Idle Recovery Dialog overlay.

- **Acceptance Criteria:**
- The primary dashboard never displays more than one actionable task at a time.
- Zero user input for 300 seconds successfully pauses the active session and triggers `session.is_suspended == true`.

- **Suggested Git Branch:** `feat/m6-action-hub`

## Milestone 7: LLM Decision Pipeline

**Objective:** Connect the cloud LLM to select the optimal next task, wrapping it in strict sandboxing.

- **Relevant PDS Chapters:** 6.1 (Prompt Construction), 6.2 (LLM Selection Logic), 6.3 (Output Constraints).
- **Dependencies:** Milestone 6.
- **Deliverables:**
- JSON payload constructor (combining state inference, filtered tasks, and user profile).
- LLM API integration module.
- Deterministic JSON parsing and local failover mechanism.

- **Acceptance Criteria:**
- LLM outputs are strictly validated against a predefined schema.
- If the LLM times out or hallucinates JSON, the UI smoothly falls back to the M5 Local Engine with no visual errors for the user.

- **Suggested Git Branch:** `feat/m7-llm-pipeline`

## Milestone 8: Onboarding & Initialization

**Objective:** Implement the first-time user initialization flow to resolve the cold-start problem and educate the user.

- **Relevant PDS Chapters:** 10.1 - 10.4 (Onboarding and Guided Tutorial System).
- **Dependencies:** Milestone 7.
- **Deliverables:**
- First-Time Activation Prompt (Modal).
- Logic to write the provisional archetype vector (`W_survey`).
- Yellow tooltip layout engine with 65% dark canvas background mask.
- 4-Step linear walkthrough sequence.

- **Acceptance Criteria:**
- Survey seamlessly writes provisional archetype data to bypass the 14-day telemetry requirement.
- Walkthrough correctly targets layout elements (Action Hub, Vault, Analytics) in the precise order specified in Chapter 10.3.

- **Suggested Git Branch:** `feat/m8-onboarding`

## Milestone 9: Time Security & Edge Case Hardening

**Objective:** Secure the absolute timeline of the application to prevent data corruption via device clock manipulation.

- **Relevant PDS Chapters:** 13.2 (Clock Tampering and Timezone Security).
- **Dependencies:** Milestone 8 (Can be done in parallel once M4 is stable).
- **Deliverables:**
- Monotonic clock validation wrapper for database writes.
- Network Time Protocol (NTP) fallback service.
- Timezone shift listener and debounce logic.

- **Acceptance Criteria:**
- Attempting to write a log with a backdated timestamp is blocked and corrected via NTP.
- Rapid timezone shifts (under 1 hour) do not duplicate task generation cycles.

- **Suggested Git Branch:** `feat/m9-time-security`
