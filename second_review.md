## Please note that the project descriptions below provide a general overview of each milestone's scope. We'll finalize the detailed, fact-checked deliverables (similar to those created for the first review) once we've selected 1-2 project proposals to pursue for GSoC.

<br>
<br>
<br>

────────────────────────────────────────────────────────────
Project A — CVE Detection & Validation Pipeline (350 hours)
────────────────────────────────────────────────────────────

**Project title**  
CVE Detection & Validation Pipeline — BLT (Buttercup/Webhook → NVD → GHSC)

**One-line abstract**  
Build an opt-in, auditable pipeline to discover CVE opportunities from scanner output and GitHub activity, validate against NVD, deduplicate/score findings, and expose pending contributions via a verification UI and REST API for downstream workflows.

---

## Background & motivation

BLT needs a reliable canonical feed of security “opportunities” so maintainers and contributors can responsibly triage and remediate unpatched CVEs.

This project implements:

- **Detection**: from GitHub webhooks (PRs/issues/reviews) and scanner output (e.g., Buttercup).
- **Validation**: NVD-based validation of CVEs and severity.
- **Safe storage**: normalized, deduplicated GHSC entries.
- **Governance**: maintainer opt‑in, rate‑limiting, and non‑intrusive behavior.
- **Verification surface**: a human‑driven verification dashboard and REST API.

It is strictly **post‑disclosure**: it only tracks contributions to already public CVEs, aligning with responsible disclosure and avoiding storage of raw exploit details.

---

## Technical approach

- **GitHubSecurityContribution (GHSC) model**
  - Django model capturing:
    - `cve_id`, `cve_score`, `severity`
    - `repo`, `contributor`, `pr_url`/issue/review linkage
    - `detected_by` (webhook vs scanner)
    - `verification_status` (pending/verified/rejected/disputed)
    - timestamps (`created_at`, `verified_at`), `verifier_id`, notes
  - Indexed for lookups by `cve_id`, `repo`, contributor, status.

- **CVE extractor**
  - Regex + heuristics to extract CVE IDs from:
    - Merged PR titles/bodies/commit messages.
    - Scanner (Buttercup) payloads.
  - Conservative matching to minimize false positives.
  - Reusable utility with unit tests on fixture PRs.

- **NVD integration**
  - Integrate via `nvdlib` (or minimal client) to:
    - Validate CVE IDs exist.
    - Fetch CVSS base scores and map severity (None/Low/Medium/High/Critical).
  - Caching layer:
    - Per‑CVE cache with configurable TTLs.
    - Backoff and retry behavior to respect NVD rate limits.
    - Offline fixtures for CI and development.

- **Opt‑in registration & configuration**
  - Per‑repo registration UI/API:
    - Maintainers explicitly opt in.
    - Configure:
      - Which events are considered (e.g., merged PRs with specific labels).
      - Scan cadence and Buttercup integration.
  - Stored config is used to drive ingestion and filter which contributions become GHSC entries.

- **Rate‑limits & duplicate detection**
  - Cache‑backed monthly caps per contributor (default: 3 verified claims/month).
  - Duplicate detection:
    - Same `cve_id + repo + pr_url` considered duplicate by default.
    - Additional checks using commit hashes or diff metadata.
  - Pre‑screen filters for doc‑only or trivial PRs (basic diff heuristic).

- **Verification Dashboard**
  - `SecurityContributionVerificationView`:
    - List of GHSC entries with `verification_status='pending'`.
    - Details:
      - CVE metadata (from NVD cache).
      - Links to PR/issue/review diff on GitHub.
      - Contributor + repo context.
    - Actions:
      - Verify, Reject, Dispute.
      - Optional notes and tags.
    - Audit trail:
      - Who verified, when, and with what comment.

- **REST API & events**
  - Endpoints:
    - `GET /api/security-contributions` (filterable list).
    - `GET /api/security-contributions/{id}` (detail).
    - `POST /api/security-contributions/{id}/verify` (for maintainers).
  - On verification:
    - Emit a Django signal (internal consumers).
    - Optionally emit a webhook event (future external consumers, e.g., rewards/education).

---

## Concrete deliverables

- **Data model & plumbing**
  - `GitHubSecurityContribution` Django model + migration + indices.
  - CVE extractor utility with tests.
  - Buttercup ingestion adapter.

- **Services**
  - NVD service using `nvdlib` with caching and CI fixtures.
  - Webhook ingestion wiring (PRs/issues/reviews) + opt‑in registration UI/API.

- **Controls & UX**
  - Rate‑limit and duplicate detection hooks.
  - Verification Dashboard (maintainer/verifier UI).
  - REST API: list, detail, verify.

- **Quality & docs**
  - Unit, integration, and end‑to‑end tests (merged PR → GHSC → verify).
  - Demo script for maintainers.
  - Developer & maintainer docs, deployment notes.
  - Pilot checklist and pilot report for 2–3 opt‑in repos.

---

## 12‑week milestone plan

- **Weeks 0–2 (Community bonding)**
  - Finalize design and GHSC schema with mentors.
  - Prepare NVD fixtures and GitHub/Buttercup fixtures.
  - Recruit pilot maintainers and confirm 2–3 pilot repos.

- **Weeks 3–4 (M1)**
  - Implement GHSC model and migration.
  - Implement basic CVE extractor and tests.
  - Run extractor on fixture PRs/issues.

- **Weeks 5–6 (M2)**
  - Wire webhook ingestion to GHSC creation.
  - Implement Buttercup ingestion adapter.
  - Build opt‑in registration UI/API for repos.
  - **Midterm target**: GHSC rows are being created from fixture events for registered repos.

- **Weeks 7–8 (M3)**
  - Integrate NVD service with caching and backoff.
  - Map CVSS to severity buckets and add to GHSC.
  - Add integration tests for invalid CVEs and API failure scenarios.

- **Weeks 9–10 (M4)**
  - Implement rate‑limit enforcement and duplicate detection heuristics.
  - Add doc‑only PR detection heuristic.
  - Extend tests to cover spammy or repetitive claims.

- **Week 11 (M5)**
  - Build Verification Dashboard & permissions.
  - Implement REST API (list/detail/verify).
  - E2E tests: merged PR → GHSC → pending UI → verify via dashboard/API.

- **Week 12 (M6)**
  - Run pilot with 2–3 opt‑in repos.
  - Collect metrics and feedback, tune heuristics.
  - Finalize docs, demo video, and pilot report.

---

## Midterm evaluation (≈ week 6)

- GHSC model implemented and migrated.
- CVE extractor passes unit tests on fixture PRs.
- Webhook ingestion and opt‑in registration working end‑to‑end in dev.
- NVD client integrated with mocked/fixture‑backed CI support.

---

## Final evaluation (≈ week 12)

- End‑to‑end demo:
  - Merged PR/scan result →
  - GHSC pending →
  - Maintainer verification →
  - GHSC verified + event emitted.
- REST API confirmed for list/detail/verify.
- Tests cover core flows and fraud scenarios.
- Pilot report summarizing:
  - Opt‑in repos,
  - Number of GHSC entries,
  - False‑positive rate,
  - Maintainer response times.

---

## Tests & fraud scenarios

- Fake/invalid CVEs (NVD no‑match).
- Duplicate claims (same `cve_id + repo + pr_url`).
- Doc‑only PRs (very small/no code diff).
- Rate‑limit breaches and throttling behavior.
- NVD downtime fallback using cached fixtures.

---

## Dependencies

- BLT webhook handlers for GitHub events.
- Buttercup (or similar) scanner output format (or stubbed adapter).
- `nvdlib` for NVD API integration.
- Redis or Django cache backend for rate‑limits and caching.

---

## Metrics (for GSoC/evaluation)

- Number of opt‑in repos (pilot target: ≥ 3).
- GHSC records created per scan/merge cadence.
- False‑positive rate (pilot target: < 15%).
- Median advisory/merge → maintainer verification time (target: < 72 hours).
- Unit/integration test coverage for the GHSC module.

---

## Risks & mitigations

- **NVD rate limits / downtime**: caching, exponential backoff, CI fixtures.
- **False positives**: conservative heuristics; human verification required before any downstream action.
- **Privacy / exploit leakage**: store only public, non‑sensitive fields; never store exploit payloads or private advisory content.

---
<br>
<br>
<br>
<br>
<br>

────────────────────────────────────────────────────────────
Project B — Security Contribution Gamification & Recognition (350 hours)
────────────────────────────────────────────────────────────

**Project title**  
Security Contribution Gamification & Recognition — BLT (Rewards, Badges, Leaderboards & Challenges)

**One-line abstract**  
Consume verified GHSC events to award BACON/badges (idempotently and auditable), maintain contributor reputation/tiers, provide leaderboards and security challenges, and offer analytics and admin audit tools.

---

## Background & motivation

Once GHSC verified records exist (from Project A or an equivalent pipeline), BLT needs to convert those verified remediation efforts into sustainable, trustworthy incentives:

- Reward tokens (BACON),
- Badges and reputation tiers,
- Leaderboards and security challenges,
- Analytics and audit tooling to detect and mitigate abuse.

This project implements the **“Project B” layer**: it does **not** do detection or NVD work, but rather assumes a feed of **verified** security contributions and focuses on **rewards, recognition, and governance**.

---

## Technical approach

- **Reward signal handlers**
  - Django signal listeners (or webhook consumers) for GHSC verification events.
  - Award BACON via existing `giveBacon()` (or log simulated payouts during pilot).
  - Record all rewards in a **payout ledger** with:
    - `ghsc_id`, `user_id`, award type (BACON, badge), amount, timestamps.
  - Enforce idempotency using **unique constraints** on `(ghsc_id, award_type)`.

- **Badges & reputation**
  - Define badge rules (examples):
    - First CVE fix.
    - First Critical CVE fix.
    - 5/10/20 verified CVE fixes.
  - Implement:
    - Reputation points ledger (per user),
    - Tiers (Beginner / Intermediate / Trusted).
  - Integrate with existing `Badge`/`UserBadge` models where possible.

- **Security leaderboards**
  - Implement `SecurityLeaderboardView` + service:
    - Severity‑weighted score (e.g., Critical > High > Medium > Low).
    - Filters by time range (last 30 days / year / all‑time).
    - Breakdown by repo, org, and individual contributor.
  - REST API:
    - `GET /api/security-leaderboard` with query parameters for period, scope.
    - Optional CSV/JSON export for reports.

- **Security challenges**
  - Add security challenge types to existing `Challenge` model:
    - E.g., “Fix 3 Medium+ CVEs this month”, “Review 5 security PRs”.
  - Hook GHSC verification events into `challenge_signals.update_challenge_progress`.
  - Ensure idempotent progress updates and clear completion rules.

- **Admin audit & analytics**
  - Admin UI to:
    - View payout history and badge issuance.
    - Inspect and resolve disputes.
    - Revoke or annotate problematic awards.
  - Analytics dashboard:
    - Payouts over time, by severity and repo.
    - Top contributors and distribution of reputation tiers.

- **Fraud prevention**
  - Idempotency at award level (no double‑pay).
  - Monthly caps on rewards per contributor.
  - Simple heuristics (and manual review) to spot:
    - Repetitive trivial changes,
    - Suspicious claim patterns.

---

## Concrete deliverables

- **Core logic**
  - Reward handlers (Django signal listeners) for GHSC verification events.
  - Payout ledger model with idempotency guarantees.
  - Badge engine (rules + reputation system).

- **User‑facing features**
  - `SecurityLeaderboardView` + API endpoints.
  - Security challenge definitions wired into `challenge_signals`.
  - Analytics dashboard and admin payout/audit UIs.

- **Quality & docs**
  - Tests for:
    - Idempotency and duplicate events.
    - Disputes and revocation flows.
    - Leaderboard aggregation correctness.
  - Documentation:
    - Contributor guide (how rewards work).
    - Verifier guide (impact of verification).
    - Steward policy for payouts and revocations.

---

## 12‑week milestone plan

- **Weeks 0–2 (Bonding)**
  - Finalize BACON weights, badge criteria, and reputation tier thresholds.
  - Confirm GHSC verification event schema (from Project A or mocks).

- **Weeks 3–5 (M1) — Reward plumbing**
  - Implement GHSC verification event consumer (signal/webhook).
  - Design and implement payout ledger model + unique constraints.
  - Implement idempotent BACON/badge awards.
  - Add unit tests for event replays and duplicate handling.

- **Weeks 6–7 (M2) — Badges & reputation**
  - Implement badge rules and reputation points ledger.
  - Integrate with `Badge`/`UserBadge` where possible.
  - Tests for badge thresholds and tier transitions.

- **Weeks 8–9 (M3) — Leaderboards**
  - Implement `SecurityLeaderboardView` and APIs.
  - Severity‑weighted ranking and time‑window filters.
  - Basic export (CSV/JSON) support.
  - Tests for ranking, pagination, and filters.

- **Week 10 (M4) — Challenges**
  - Define security challenge types.
  - Integrate GHSC verification events with `challenge_signals`.
  - Tests for challenge tracking and completion.

- **Weeks 11–12 (M5) — Analytics & audit**
  - Build admin payouts/audit views and basic analytics dashboard.
  - Implement dispute and revocation workflows.
  - Final docs and an end‑to‑end demo scenario.

---

## Midterm evaluation

- Reward handlers implemented and covered by idempotency tests.
- Badge issuance and reputation tiering working in unit tests.
- Prototype leaderboard API available and returning meaningful data (even if mocked).

---

## Final evaluation

- Full demo:
  - GHSC verification event →
  - BACON/badge awards logged →
  - Leaderboard and challenges updated →
  - Visible in analytics/audit UIs.
- Admin audit interface shows payout history, disputes, and revocations.
- Tests cover fraud scenarios, idempotent behavior, and leaderboard correctness.

---

## Tests & fraud scenarios

- Duplicate GHSC verification events **do not** double‑award tokens.
- Rejected GHSC records **do not** produce any awards.
- Revocation or dispute:
  - Removes or marks awards.
  - Recomputes leaderboard and reputation as needed.
- “Reputation gaming” tests:
  - Many low‑impact PRs vs. fewer high‑impact ones.
  - Monthly caps and tier gating enforced.

---

## Dependencies

- GHSC verification event schema and endpoints (from Project A or a mocked feed).
- Existing BLT `giveBacon()` function and badge models.
- BLT challenge and leaderboard primitives.

---

## Metrics

- Number of verified contributions awarded (pilot target: ≥ 2).
- Leaderboard engagement (active contributors, page views).
- Badge issuance counts and distribution.
- Dispute/revocation counts (should be low).

---

## Risks & mitigations

- **Reward abuse**: monthly caps, reputation gating, and manual steward oversight.
- **Idempotency bugs**: strong DB constraints and transactional award records.
- **BACON economics**: start with simulated or logged payouts during pilot to validate behavior before “real” rewards.

---

<br>
<br>
<br>
<br>
<br>

────────────────────────────────────────────────────────────
   Why Combining Project A + Project B into One 350‑Hour Project Is Impractical
────────────────────────────────────────────────────────────

**Context**  
Projects A and B describe two distinct but related systems:

- **Project A**: CVE Detection & Validation Pipeline — GHSC model, NVD validation, dedupe/rate‑limits, opt‑in governance, verification UI/API.
- **Project B**: Security Contribution Gamification & Recognition — rewards, badges, leaderboards, challenges, analytics and fraud controls, all built on verified GHSC events.

This section explains why combining **both A and B** into a single 350‑hour GSoC project is **not recommended**, and why splitting them leads to safer, higher‑quality outcomes.

---

## Concise rationale

- **Scope duplication & complexity**
  - **Project A** alone includes:
    - Schema design and migrations,
    - NVD integration and caching,
    - Duplicate and spam filtering,
    - Opt‑in registration and verification UI.
  - **Project B** alone includes:
    - Idempotent reward plumbing,
    - Badge and reputation logic,
    - Leaderboards, challenges, admin analytics, and fraud prevention.
  - Implementing both to production quality in one 350‑hour slot would require cutting corners on either detection safety (A) or reward correctness (B).

- **Safety & trust cost**
  - Errors in detection (false positives, noisy or low‑quality GHSC entries) and errors in rewards (double payouts, mis‑awards) **directly damage maintainer trust**.
  - Both systems require:
    - Iterative pilots,
    - Careful tuning of heuristics,
    - Time to gather real feedback.
  - Rushing both at once increases the risk of shipping something fragile in both layers.

- **Integration & testing burden**
  - End‑to‑end testing of:
    - detection → verification → reward → leaderboard/challenge
    - requires:
      - Extensive fixtures and test harnesses,
      - Real maintainers for pilot verification,
      - Time to analyze pilot metrics and refine policies.
  - Packing that full stack into one project compresses design, implementation, and iteration too much.

- **Mentorship overhead**
  - A combined A+B project demands:
    - Frequent design reviews,
    - Deep code reviews across backend, APIs, UI, and reward logic,
    - Tight coordination on pilot operations.
  - This can exceed realistic mentor bandwidth for a single student’s 350‑hour slot.

---

## Practical alternative

- **Split into two 350‑hour projects**:
  - **Project A (Pipeline)**:
    - Deliver a robust, opt‑in GHSC and NVD‑backed detection + verification surface.
    - Ends with stable, verified events and APIs.
  - **Project B (Gamification & Recognition)**:
    - Consume verified GHSC events to build rewards, badges, leaderboards, and challenges.
    - Can start with mocked events if needed, then integrate with A.

This separation:

- Reduces risk to maintainers and contributors.
- Allows each student (or iteration) to focus deeply on one problem space.
- Aligns with the 2026 direction of first establishing a reliable vulnerability/remediation pipeline, then layering incentives and education on top.

---

## If a compressed combined MVP is required

If mentors strongly prefer a **single combined A+B MVP**, the project would need very strict scope reductions, such as:

- **For Project A**:
  - Minimal GHSC fields and a limited subset of event sources (e.g., only merged PRs with labels).
  - Simplified NVD integration using fixtures and basic caching.
  - Basic verification UI without advanced filters or bulk actions.

- **For Project B**:
  - Rewards as **logged simulations only** (no real BACON transfers in production).
  - A single, simple severity‑weighted leaderboard (no advanced filters).
  - No challenges or complex reputation tiers; just base badges and a basic scoreboard.
  - Very limited pilot (1 repo, few contributors).

Even under this constrained MVP, much of the fraud prevention, governance, and production‑hardening work would remain as **future work**, making it less suitable as a “complete” GSoC deliverable.

For these reasons, the recommended plan is to **treat A and B as two separate 350‑hour projects**, each with its own milestones, tests, and pilot.

<br>
<br>
<br>
<br>
<br>

────────────────────────────────────────────────────────────
Alternative (preferred) — Project B + Light C (Single 350‑Hour Project)
Security Rewards & Leaderboards + Education Bridge
────────────────────────────────────────────────────────────

**Project title**  
Security Rewards & Leaderboards + Education Bridge (B + light C)

**One-line abstract**  
Deliver the rewards, badges, reputation, leaderboards, challenges, and analytics (Project B) and include a lightweight education bridge API so blt‑education teams can consume badges/leaderboards to unlock courses — without delivering full curriculum content.

---

## Why this option

- Provides **immediate community value**:
  - Recognizes verified security contributions with rewards and leaderboards.
  - Encourages sustained contributor growth.
- Introduces a **lightweight bridge to education**:
  - Allows blt‑education to consume badge/leaderboard data for course unlocks.
- Keeps the overall scope **realistic for a single 350‑hour project**:
  - No full curriculum, labs, or auto‑grading engine.
  - Focuses on infrastructure and APIs that education teams can build on.
- Can be implemented with:
  - Real GHSC events from Project A, or
  - Mocked verification events if Project A is not yet deployed.

---

## Technical approach

- **Rewards & recognition (Project B core)**
  - Reward handlers subscribed to GHSC verification events.
  - BACON awards via `giveBacon()` (or simulated during pilot).
  - Badge and reputation system with tiers (Beginner, Intermediate, Trusted).
  - Severity‑weighted `SecurityLeaderboardView` and APIs.
  - Security challenges integrated with `challenge_signals`.
  - Admin analytics and payout/audit UIs.
  - Fraud controls: idempotent awards, caps, basic heuristics.

- **Education bridge (light C)**
  - Read‑only **education-facing APIs** and/or webhooks:
    - **Badge/leaderboard queries**:
      - Endpoint to list a user’s badges and reputation tier.
      - Endpoint to fetch leaderboard positions or scores.
    - **Badge event webhook**:
      - Optional outbound event when a badge or tier is earned.
  - These APIs allow:
    - blt‑education to unlock courses based on badges/tier.
    - Education dashboards to highlight top security contributors.
  - Critical constraint:
    - Education bridge endpoints **do not expose any raw vulnerability details**.
    - Only aggregate or identity‑level data (badges, tiers, scores) is shared.

---

## Concrete deliverables

- **Rewards & recognition**
  - Reward handlers and idempotent payout logs.
  - Badge and reputation engine (models + APIs).
  - `SecurityLeaderboardView` + REST API.
  - Security challenges integration with `challenge_signals`.
  - Analytics dashboard and admin payout/audit UI.

- **Education bridge**
  - Read‑only REST endpoints for:
    - Badge/reputation lookups.
    - Leaderboard lookups.
  - Optional webhook to notify blt‑education on badge/tier achievements.
  - Documentation for education teams:
    - How to query badges/leaderboards.
    - How to wire badge events to course unlocks.

- **Testing & docs**
  - Tests and mocked demos using GHSC verification events (real or simulated).
  - Contributor guide (how security rewards work).
  - Education integration guide (how to consume the bridge).

---

## 12‑week milestone plan

- **Weeks 0–2 (Bonding)**
  - Define badge and reputation rules.
  - Agree on “education contract”:
    - Which fields will be exposed to blt‑education.
    - Which events trigger education webhooks.
  - Confirm whether GHSC events are real or mocked.

- **Weeks 3–5**
  - Implement reward plumbing & idempotency tests (Project B core).
  - Implement payout ledger, BACON awards, and basic badges.
  - Unit tests for duplicate events and simulated pilot flows.

- **Weeks 6–7**
  - Implement full badge & reputation system (tiers).
  - Tests for tier transitions and badge issuance.
  - Initial SecurityLeaderboardView and APIs.

- **Weeks 8–9**
  - Refine leaderboard & challenge integration.
  - Implement security challenges and test progress tracking.
  - Basic analytics dashboard.

- **Weeks 10–11**
  - Implement the **education bridge endpoints/webhooks**:
    - Badge/leaderboard query APIs.
    - Event webhook for badge/tier changes (if agreed).
  - Write integration docs and simple mock demos for education teams.

- **Week 12**
  - Pilot demo using mocked or real GHSC events.
  - Final docs and end‑to‑end walkthrough (verification event → reward → leaderboard → education endpoint).

---

## Uncovered points in “light C” (explicitly out of scope)

To keep scope realistic, the following **are not included** and would belong to a future, larger “education platform” project:

- Content creation: labs, playbooks, exercises.
- Auto‑grading engine and instructor review queue.
- Mentor/cohort administration tools or scheduling.
- Curriculum governance and redaction review pipelines.

The education bridge here is **infrastructure only**: it enables education teams to consume security contribution signals without imposing full education platform responsibilities into this project.

---

## Acceptance criteria & metrics

- Verified (or mocked) GHSC events:
  - Trigger reward logic idempotently.
  - Update badges and reputation tiers.
  - Reflect in security leaderboards and challenges.
- Education bridge endpoints:
  - Correctly expose badges/tiers and leaderboard positions.
  - Can be used by a small example client (mock blt‑education integration).
- Metrics:
  - Number of badges/tier changes visible through the bridge.
  - Leaderboard updates after events.
  - Basic logs showing education endpoint usage.

---

## Risks & mitigations

- **Coupling to Project A**:
  - Use mocked verification events if A is not ready.
  - Define GHSC event schema early and code to that contract.
- **Education misuse**:
  - Education endpoints only expose badge/leaderboard data.
  - No raw vulnerability descriptions or exploit content.

<br>
<br>
<br>
<br>
<br>

────────────────────────────────────────────────────────────
Decoupling Project B from Project A (Parallel Development)
────────────────────────────────────────────────────────────

To keep Project B fully viable as an independent GSoC project (and allow it to run in parallel with Project A), I will design it around a generic **VerifiedSecurityContribution** event schema instead of a concrete GHSC model.

**Abstract event schema**

Project B will consume a simple, abstract event type:

- `id`
- `user_id`
- `severity`
- `cve_id` (optional)
- `repo`
- `verified_at`
- `source` (e.g., “manual”, “scanner”, “GHSC”)

Any system that can emit events in this shape can drive Project B. Project A (GHSC pipeline) is **one possible producer**, but not the only one.

**Event adapter inside Project B**

Project B will include a thin “event adapter” that:

- Accepts `VerifiedSecurityContribution` payloads (from DB, signals, webhooks, or fixtures),
- Normalizes them to B’s internal representation,
- Triggers rewards, badges, leaderboards, and challenge updates.

For GSoC, this adapter will read from:

- Fixtures (seeded verified contributions),
- A minimal `VerifiedSecurityContribution` table,
- Or a small admin UI where mentors can manually mark contributions as verified.

**Fixtures and admin tools as initial source**

During GSoC, Project B does **not** require GHSC or Project A to be finished:

- I will seed the database with fixture “verified contributions”, and/or
- Provide a simple admin form: “Create verified security contribution”.

All milestones and demos for B will use these contributions to:

- Trigger reward handlers,
- Update leaderboards and challenges,
- Drive analytics and admin views.

**A → B integration as an optional add‑on**

In the proposal and implementation, I will explicitly treat A→B integration as **optional**:

> During GSoC, I will implement Project B against a generic `VerifiedSecurityContribution` feed (fixtures/admin UI/minimal table). If Project A’s GHSC pipeline is available, I will add a thin adapter that maps GHSC → `VerifiedSecurityContribution`, but that integration is optional and can land late or after GSoC.

This design ensures:

- Project B is fully demo‑able with synthetic or manually created verified contributions.
- Projects A and B can be developed **in parallel by different students**.
- After GSoC, maintainers can plug A’s GHSC events into B’s adapter without changing B’s core logic.

<br>
<br>
<br>
<br>
<br>

────────────────────────────────────────────────────────────
Combined Project C + D (Single 350‑Hour Project)
blt‑education + Knowledge Sharing & Community Impact
────────────────────────────────────────────────────────────

**Project title**  
blt‑education + Knowledge Sharing Platform — Hands‑on Labs, Safe Reports & Playbooks

**One-line abstract**  
Create a standalone blt‑education platform with tiered hands‑on learning tracks and labs, auto‑quiz & instructor review flows, and a knowledge‑sharing pipeline that produces safe, anonymized dashboards and templated reports and playbooks derived from BLT data.

---

## Why combine C + D

Education and knowledge‑sharing rely on the same:

- **Data sources**: aggregated, sanitized contribution and vulnerability/fix data.
- **Governance**: redaction, privacy, and approval workflows.

Combining them into one project:

- Reduces duplication across “education” and “community impact” tooling.
- Creates a coherent curriculum informed by aggregated insights.
- Provides safer, well‑reviewed public outputs (dashboards, playbooks, case studies).

This project assumes BLT already has a way to track verified security contributions (e.g., GHSC), but it does **not** depend on a particular implementation; it can work with mocked or generic fixtures if needed.

---

## Concrete deliverables

- **blt‑education scaffold**
  - New repository or module with:
    - Project scaffold and CI.
    - Contribution guide and coding standards.
  - Basic UI shell (Tailwind-based) for student and instructor views.

- **Tiered learning tracks & labs**
  - Learning tracks: **Beginner → Intermediate → Trusted**.
  - Implement 6–10 hands‑on labs:
    - Focus on safe, synthetic scenarios (no live vulnerabilities).
    - Each lab includes:
      - Instructions,
      - Sandbox or code snippet,
      - Quiz or short exercise.

- **Auto‑quiz engine & review queue**
  - Auto‑quiz engine for basic checks (multiple‑choice, simple code analysis).
  - Instructor review queue for:
    - Subjective assessments,
    - Lab submissions that need human feedback.
  - Feedback UI for instructors to respond to students.

- **Education ↔ BLT integration endpoints**
  - Read‑only integration with BLT:
    - Query badges/reputation/leaderboards (if available).
    - Webhook subscription for “learner reached Trusted tier” or similar.
  - Optional: unlock advanced labs or tracks based on BLT signals (e.g., Verified security contributor).

- **Anonymization & aggregation pipeline**
  - Pipeline to:
    - Aggregate data from BLT (e.g., counts of CVEs fixed by severity, per language).
    - Anonymize and sanitize:
      - No repository or user PII unless explicitly permitted.
      - No raw vulnerability or exploit content.
  - Output:
    - Aggregated datasets for dashboards and reports.

- **Dashboards & report generator**
  - Dashboards:
    - High‑level trends (e.g., common vulnerability types, fix frequencies).
    - Educational insights (e.g., which lab topics map to common real‑world issues).
  - Monthly or quarterly report generator:
    - Templated “state of fixes” reports.
    - Includes charts and anonymized stats.

- **Playbooks & remediation templates**
  - A small set (e.g., 3–5) of remediation playbooks:
    - Sanitized, pattern‑based guidance (e.g., “fixing typical XSS configuration issues in Django”).
  - Two‑person approval workflow for publishing case studies:
    - Ensure no sensitive details are leaked.
    - Sign‑off from both a technical reviewer and privacy/security reviewer.

- **Mentor/instructor tools & pilot**
  - Simple tools for:
    - Assigning labs to a pilot cohort (5–10 students).
    - Viewing progress and lab completion.
  - Run a small pilot and capture feedback in a final pilot report.

---

## 12‑week milestone plan

- **Weeks 0–2 (Bonding & design)**
  - Decide repo structure and tech stack for blt‑education.
  - Define redaction and privacy policy with mentors.
  - Recruit pilot cohort (5–10 students) and a small instructor group.

- **Weeks 3–4**
  - Implement repo scaffold and CI.
  - Build 1 end‑to‑end beginner lab:
    - Lab instructions,
    - Simple quiz or exercise,
    - Auto‑grader path.
  - Basic student UI to start/complete the lab.

- **Weeks 5–6**
  - Implement:
    - Auto‑quiz engine,
    - Instructor review queue,
    - Feedback UI.
  - Add 1–2 more labs and tests.

- **Weeks 7–8**
  - Implement BLT integration endpoints:
    - Badge/leaderboard queries (if available).
    - Webhook subscription for relevant events.
  - Add basic mechanics to unlock advanced labs or tracks based on BLT signals.

- **Weeks 9–10**
  - Build anonymization & aggregation pipeline:
    - Ingest synthetic or fixture BLT data.
    - Produce anonymized aggregates.
  - Implement dashboards and a report generator (e.g., a monthly “education impact” report).

- **Weeks 11–12**
  - Draft a few remediation playbooks and case‑study templates.
  - Implement two‑person approval workflow for publishing.
  - Run pilot with 5–10 students.
  - Capture results in a final pilot report and refine documentation.

---

## Midterm & final evaluation

- **Midterm**
  - At least one sandboxed lab with:
    - Auto‑quiz,
    - Basic grading.
  - Education API stub demonstrating:
    - Example of unlocking a lab based on a mocked badge or tier.

- **Final**
  - At least 3 labs completed and working end‑to‑end.
  - Auto‑grading + instructor review flow functional.
  - Badge/leaderboard integration for unlocking advanced content.
  - An anonymized dashboard and at least one generated safe report.
  - Pilot report summarizing:
    - Cohort experience,
    - Main metrics,
    - Lessons learned.

---

## Metrics

- Lab completion rate (pilot target: ≥ 60%).
- Time‑to‑feedback for reviewed labs (target: < 72 hours).
- Number of badge‑driven unlocks and course claims (if integrated with BLT).
- Number of anonymized reports produced and dashboard views.
- Zero privacy incidents or policy violations.

---

## Risks & mitigations

- **Privacy / exploit disclosure**
  - Strict redaction rules and technical controls.
  - Two‑person approval for any case study/playbook referencing BLT data.
- **Content maintenance**
  - Start with a small, high‑quality set of labs and playbooks.
  - Use clear contribution guidelines to accept community improvements.
- **Dependency on Project A/B**
  - Design integration to work against mocks and a stable API contract.
  - Ensure labs and dashboards are meaningful even without live data.

---
