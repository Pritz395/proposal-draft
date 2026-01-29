# Second Review — Strategic Options & Project Portfolio

## Purpose and scope
This document synthesizes community input from GitHub Discussion #5495 and defines a strategic direction for BLT in 2025-2026. It outlines a portfolio of four potential GSoC projects (A-D), their tradeoffs, and how they integrate with existing infrastructure.

This document supersedes and/or extends informal planning notes and chat-based decisions.

## Executive summary
The community direction is remediation-first: find unpatched vulnerabilities, notify maintainers with high-quality signals, and reward verified fixes. The four project options are complementary but each must stand alone within a 350-hour GSoC slot.

**Recommendation (largest impact within timeframe):** **Project B + light C.** It delivers immediate, visible community value (rewards, leaderboards, challenges) while adding a minimal education bridge that enables future coursework without the full education platform scope. It is feasible in 350 hours and can run with mocked or manual verified events if Project A is not yet live.

**Alternate foundation-first path:** If mentors want to establish the pipeline first, **Project A** is the best starting point because it creates the verified contribution signal for downstream rewards and education.

**Personal interest:** I am most interested in **Project B + light C** as a single 350-hour project. The core rewards/leaderboards infrastructure (B) is my primary focus, with light C providing a minimal education bridge through read-only APIs or a simple webhook that expose existing B outputs. This keeps the scope aligned with what maintainers care about while adding a small bridge value without content creation overhead.

## Strategic Direction (2025-2026)
- **Remediation-first security:** focus on verified fixes, not raw report volume.
- **Balance UI/UX and security:** separate workflows and clear severity labeling.
- **Maintainer notification + fix workflows:** opt-in scanning, actionable alerts, verification trail.
- **Leverage existing capabilities:** AI tooling, BACON system, organization management.
- **Align with GSoC focus areas:** AI-assisted scoring, interactive security labs, org dashboards.

## Project options overview (A / B / C / D)
- **Project A:** CVE Detection & Validation Pipeline (GHSC model, NVD validation, verification UI/API).
- **Project B:** Security Contribution Gamification & Recognition (BACON, badges, leaderboards, challenges).
- **Project C:** blt-education platform (labs, quizzes, instructor review workflows).
- **Project D:** Knowledge sharing & community impact (anonymized reports, dashboards, playbooks).
- **light C:** A minimal education bridge (read-only APIs/webhooks for badges/leaderboards), not a full education platform.

## Differentiation at a glance
Project | Focus | Primary beneficiaries | Dependencies | Risk level
---|---|---|---|---
A | Detection + validation | Maintainers, contributors | NVD API, scanning | High (false positives)
B | Rewards + recognition | Active contributors | Verified signals (real or mocked) | Medium (gaming, economics)
C | Education platform | New contributors | Content + mentoring capacity | Medium (content burden)
D | Knowledge sharing | OSS ecosystem | Aggregated data + governance | Medium (privacy controls)

## Decision guide
- **Immediate contributor engagement:** choose **Project B + light C**.
- **Strong education/content team available:** consider **Project C + D**.
- **Foundational pipeline needed first:** prioritize **Project A**.

## Project dependency flow
```
[GitHub PR Merge / Scanner] 
        ↓
[Project A: GHSC Pipeline] → verified events
        ↓
[Project B: Rewards Engine] → BACON, badges, leaderboards
        ↓
[light C: Education Bridge] → read-only APIs for curriculum
        ↓
[Future: Full C/D Platform] → labs, playbooks, dashboards
```
**Note:** Project B can run independently using mocked verification events or manual admin entries if Project A is not yet live.

## Notes
My preferred scope is **Project B + light C** as outlined in this document.

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

## Pros
- Foundational for all downstream rewards, badges, and education.
- Directly improves security triage and verification workflows.
- Clear alignment with responsible disclosure practices.
- Creates auditable, governed contribution signals.

## Cons
- NVD integration and verification UX add complexity.
- False positives require tuning and maintainer time.
- Less immediate user-facing impact compared to rewards/leaderboards.
- External API dependency (NVD) introduces rate-limit and uptime risks.

## Immediate and long-term benefits
**Immediate benefits:**
- Produces a verified security contribution feed with clear governance.
- Gives maintainers a verification workflow and actionable signal quality controls.

**Long-term benefits:**
- Foundation for rewards, education, and broader security analytics.
- Scalable CVE pipeline that can be extended to more repos and inputs.

---

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

## Pros
- Immediate community value (rewards, leaderboards, challenges).
- Builds on existing BLT incentives and gamification infrastructure.
- Clear demo impact even with mocked verification events.
- Directly addresses contributor motivation and retention.

## Cons
- Depends on verified contribution signals (real or mocked).
- Requires careful idempotency and anti-abuse controls.
- BACON economics need validation before real payouts.
- Gaming risk if quality controls are insufficient.

## Immediate and long-term benefits
**Immediate benefits:**
- Visible incentives (BACON, badges, leaderboards) for verified CVE work.
- Stronger contributor engagement around security fixes.

**Long-term benefits:**
- Credible reward system that reinforces quality over volume.
- Durable reputation signals for contributors and maintainers.

---

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
- `source` (e.g., "manual", "scanner", "GHSC")

Any system that can emit events in this shape can drive Project B. Project A (GHSC pipeline) is **one possible producer**, but not the only one.

**Event adapter inside Project B**

Project B will include a thin "event adapter" that:

- Accepts `VerifiedSecurityContribution` payloads (from DB, signals, webhooks, or fixtures),
- Normalizes them to B's internal representation,
- Triggers rewards, badges, leaderboards, and challenge updates.

For GSoC, this adapter will read from:

- Fixtures (seeded verified contributions),
- A minimal `VerifiedSecurityContribution` table,
- Or a small admin UI where mentors can manually mark contributions as verified.

**Fixtures and admin tools as initial source**

During GSoC, Project B does **not** require GHSC or Project A to be finished:

- I will seed the database with fixture "verified contributions", and/or
- Provide a simple admin form: "Create verified security contribution".

All milestones and demos for B will use these contributions to:

- Trigger reward handlers,
- Update leaderboards and challenges,
- Drive analytics and admin views.

**A → B integration as an optional add‑on**

In the proposal and implementation, I will explicitly treat A→B integration as **optional**:

> During GSoC, I will implement Project B against a generic `VerifiedSecurityContribution` feed (fixtures/admin UI/minimal table). If Project A's GHSC pipeline is available, I will add a thin adapter that maps GHSC → `VerifiedSecurityContribution`, but that integration is optional and can land late or after GSoC.

This design ensures:

- Project B is fully demo‑able with synthetic or manually created verified contributions.
- Projects A and B can be developed **in parallel by different students**.
- After GSoC, maintainers can plug A's GHSC events into B's adapter without changing B's core logic.

<br>
<br>

────────────────────────────────────────────────────────────
Project C — blt-education Platform (350 hours, standalone)
────────────────────────────────────────────────────────────

**Project title**  
blt-education Platform — Hands-on Labs, Quizzes, and Instructor Review

**One-line abstract**  
Build a structured security education platform with tiered learning tracks, hands-on labs, auto-quizzes, instructor review workflows, and optional BLT badge-based unlocks.

---

## Scope if standalone
- Repository scaffold and CI for blt-education (or a dedicated module).
- Tiered learning tracks: Beginner → Intermediate → Trusted.
- 4–6 high-quality hands-on labs with sandboxed exercises.
- Auto-quiz engine and instructor review queue.
- Mentor/cohort administration tools.
- Integration with BLT badges/leaderboards to unlock courses.
- Pilot with 10–20 students.

## Deliverables
- Working education platform with course progression and assessments.
- Auto-grading + manual review workflows.
- Student and instructor UIs.
- Badge-based unlock integration.
- Documentation and pilot report.

## Pros
- Structured onboarding path for new security contributors.
- Clear progression from beginner to trusted contributor.
- Reusable curriculum with community contribution potential.

## Cons
- High content creation burden for labs and assessments.
- Requires instructor/mentor availability for review queues.
- Lower immediate impact if verified contribution signals are not mature.

## Immediate and long-term benefits
**Immediate benefits:**
- Skill-building pathway for new contributors.
- Early signal of contributor readiness for security tasks.

**Long-term benefits:**
- Scalable education asset that grows the contributor pipeline.
- Shared knowledge base that improves security literacy over time.

---

<br>
<br>

────────────────────────────────────────────────────────────
Project D — Knowledge Sharing & Community Impact (350 hours, standalone)
────────────────────────────────────────────────────────────

**Project title**  
Knowledge Sharing & Community Impact — Dashboards, Reports, Playbooks

**One-line abstract**  
Create an anonymized data pipeline and public-facing insights (dashboards, reports, and remediation playbooks) without exposing sensitive vulnerability details.

---

## Scope if standalone
- Anonymization and aggregation pipeline for BLT security data.
- Public dashboards (CVE trends, severity distribution, repo categories).
- Monthly/quarterly report generator.
- Remediation playbook templates (3–5).
- Two-person approval workflow for publishing sensitive content.
- Case study generator with privacy controls.

## Deliverables
- Aggregation + anonymization service.
- Analytics dashboards (Chart.js or equivalent).
- Automated report generation.
- Playbook library with governance workflow.
- Documentation on redaction policies.

## Pros
- Broad community impact via safe public insights.
- Reusable remediation guidance without exposing sensitive details.
- Supports maintainer and founder decision-making.

## Cons
- Depends on having meaningful data to aggregate.
- Publishing workflows may be under-used without active pipelines.
- Ongoing governance overhead for redaction and approval.

## Immediate and long-term benefits
**Immediate benefits:**
- Public dashboards and reports that highlight security trends.
- Early visibility into common vulnerability patterns.

**Long-term benefits:**
- Sustainable knowledge sharing that improves ecosystem-wide hygiene.
- A trusted playbook library that scales remediation patterns.

---

<br>
<br>

────────────────────────────────────────────────────────────
Why Standalone C or D Alone May Not Be Ideal
────────────────────────────────────────────────────────────

**Project C alone:**
- High content creation burden (10–15 quality labs is substantial).
- Requires instructor/mentor availability for review queues.
- Delivers less value if verified contribution signals are not yet mature.

**Project D alone:**
- Depends on having meaningful BLT data to aggregate.
- Publishing workflows can be under-utilized without active contribution pipelines.
- Privacy/governance overhead without immediate contribution impact.

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

────────────────────────────────────────────────────────────
Alternative (personally preferred for GSoC) — Project B + light C (Single 350‑Hour Project)
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

---

## Uncovered points in “light C” (explicitly out of scope)

To keep scope realistic, the following **are not included** and would belong to a future, larger “education platform” project:

- Content creation: labs, playbooks, exercises.
- Auto‑grading engine and instructor review queue.
- Mentor/cohort administration tools or scheduling.
- Curriculum governance and redaction review pipelines.

The education bridge here is **infrastructure only**: it enables education teams to consume security contribution signals without imposing full education platform responsibilities into this project.

---

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
