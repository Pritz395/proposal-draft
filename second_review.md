# Second Review — Strategic Options & Project Portfolio

## Table of Contents
1. [Purpose and scope](#purpose-and-scope)
2. [Executive summary](#executive-summary)
3. [Community Discussion Summary](#community-discussion-summary-discussion-5495)
4. [Strategic Direction (2025-2026)](#strategic-direction-2025-2026)
5. [Project options overview](#project-options-overview-a--b--c--d)
6. [Differentiation at a glance](#differentiation-at-a-glance)
7. [Decision guide](#decision-guide)
8. [Project dependency flow](#project-dependency-flow)
9. [UI/UX consistency](#uiux-consistency)
10. [Notes](#notes)
11. **Project A** — [CVE Detection & Validation Pipeline](#project-a--cve-detection--validation-pipeline-350-hours)
12. **Project B** — [Security Contribution Gamification & Recognition](#project-b--security-contribution-gamification--recognition-350-hours)
13. [Decoupling Project B from Project A](#decoupling-project-b-from-project-a-parallel-development)
14. **Project C** — [blt-education Platform (standalone)](#project-c--blt-education-platform-350-hours-standalone)
15. **Project D** — [Knowledge Sharing & Community Impact (standalone)](#project-d--knowledge-sharing--community-impact-350-hours-standalone)
16. [Why Standalone C or D Alone May Not Be Ideal](#why-standalone-c-or-d-alone-may-not-be-ideal)
17. [Why Combining A + B Is Impractical](#why-combining-project-a--project-b-into-one-350hour-project-is-impractical)
18. **Preferred: B + light C** — [Security Rewards & Education Bridge](#alternative-preferred--project-b--light-c-single-350hour-project)
19. **Combined C + D** — [blt-education + Knowledge Sharing](#combined-project-c--d-single-350hour-project)
20. [Testing Strategy (All Projects)](#testing-strategy-all-projects)
21. [Governance & Controls](#governance--controls)
22. [Technical clarifications](#technical-clarifications-current-assumptions)
23. [Communication & Progress Tracking](#communication--progress-tracking)
24. [Documentation cross-references](#documentation-cross-references)

---

## Purpose and scope
This document synthesizes community input from GitHub Discussion #5495 and defines a strategic direction for BLT in 2025-2026. It outlines a portfolio of four potential GSoC projects (A-D), their tradeoffs, and how they integrate with existing infrastructure.

This document supersedes and/or extends informal planning notes and chat-based decisions.

## Executive summary
The community direction is remediation-first: find unpatched vulnerabilities, notify maintainers with high-quality signals, and reward verified fixes. The four project options are complementary but each must stand alone within a 350-hour GSoC slot.

**Recommendation (largest impact within timeframe):** **Project B + light C.** It delivers immediate, visible community value (rewards, leaderboards, challenges) while adding a minimal education bridge that enables future coursework without the full education platform scope. It is feasible in 350 hours and can run with mocked or manual verified events if Project A is not yet live.

**Alternate foundation-first path:** If mentors want to establish the pipeline first, **Project A** is the best starting point because it creates the verified contribution signal for downstream rewards and education.

**Personal interest:** I am most interested in **Project B + light C** as a single 350-hour project. The core rewards/leaderboards infrastructure (B) is my primary focus, with light C providing a minimal education bridge through read-only APIs or a simple webhook that expose existing B outputs. This keeps the scope aligned with what maintainers care about while adding a small bridge value without content creation overhead.

## Community Discussion Summary (Discussion #5495)
### Key themes
- Remediation-first security mission: find and fix unpatched vulnerabilities.
- Vulnerability discovery service concept: Buttercup + AI tooling integration.
- Internet-wide bug discovery for low-hanging fruit.
- BACON rewards should prioritize security-relevant PRs and verified fixes.
- Education platform expansion (blt-university concept).
- Prevent low-quality or gamed contributions.

### Personas
- **Aspiring Alex (new contributors):** wants guided learning paths and meaningful tasks.
- **Founder Frank (budget-conscious founders):** wants low-cost remediation support with credible signals.
- **Open Source Oliver (OSS maintainers):** needs actionable, low-noise notifications and opt-in control.

### Community concerns
- False positives and contribution gaming.
- Maintainer overload from noisy notifications.
- Responsible disclosure and privacy constraints.

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

## UI/UX consistency
All new views will follow BLT’s existing design patterns:
- Use the existing CSS framework and component patterns.
- Integrate with current navigation and authentication flows.
- Meet accessibility expectations (WCAG 2.1 AA where applicable).

## Notes
The project descriptions below provide detailed technical approaches, milestones, and deliverables. My preferred scope is **Project B + light C** as outlined in this document.

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

## Technical integration points
- Add `GitHubSecurityContribution` to `website/models.py` (or a new `website/security/models.py` module if we split security code).
- Extend GitHub webhook handlers in `website/views/user.py` for PR/review events.
- Add CVE/NVD logic in `website/services/cve_service.py` (or `website/services/nvd_client.py`).
- Add verification dashboard in `website/views/security.py` with templates under `website/templates/security/`.
- Register routes in `blt/urls.py` (or existing API routing module).

## API design (Project A)
- **Authentication:** Django session + CSRF for web; token-based auth for external consumers.
- **Base path:** `/api/v1/security-contributions/`.
- **Pagination:** PageNumberPagination (default 25 per page).
- **Rate limiting:** 100 requests/hour per user (via existing throttling middleware or `django-ratelimit`).
- **Filtering:** status, repo, severity, contributor via FilterBackend.
- **Errors:** consistent JSON error shape with `code`, `detail`, and `field_errors`.

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
- **Buttercup scanner output format** (or equivalent CVE detection tool).
  - **Mitigation if unavailable:** Use GitHub Security Advisory API as primary source, Dependabot alerts as fallback, or manual CVE entry via admin UI for pilot.
- `nvdlib` for NVD API integration.
- Redis or Django cache backend for rate‑limits and caching.

---

## Metrics (for GSoC/evaluation)

- Number of opt‑in repos (pilot target: ≥ 3).
- GHSC records created per scan/merge cadence.
- False‑positive rate (pilot target: < 15%).
- Median time from GHSC creation to verification UI availability (target: < 5 minutes).
- Maintainer feedback collected on verification workflow usability (target: ≥ 4/5 satisfaction).
- Unit/integration test coverage for the GHSC module.

## Metrics measurement
- **During GSoC pilot:** manual tracking in a shared sheet; target N ≥ 10 verified contributions.
- **Post‑GSoC:** automate via admin analytics and system metrics.

---

## Minimum Viable Scope (if behind schedule)

**Guaranteed minimum deliverables:**
- `GitHubSecurityContribution` model with all core fields (cve_id, repo, pr_url, contributor, severity, status) ✓
- Database migrations and admin interface ✓
- CVE extractor function with regex pattern matching and tests ✓
- GitHub webhook handler for `pull_request.closed` events ✓
- Basic verification dashboard (list view + verify/reject actions) ✓
- REST API endpoints: `GET /api/v1/security-contributions/` and `POST /api/v1/security-contributions/{id}/verify` ✓
- NVD validation with fixture-based caching (no live API during pilot) ✓
- Unit tests achieving 80%+ coverage ✓
- Basic documentation (README, API docs) ✓

**Stretch goals (may be deferred post-GSoC):**
- Buttercup scanner integration (fallback: use GitHub Advisory API + Dependabot alerts)
- Advanced duplicate detection (fallback: basic unique constraint on cve_id+repo+pr_url)
- Maintainer opt-in registration UI (fallback: admin-only repository registration)
- Severity auto-classification based on CVSS scores (fallback: manual severity entry)
- Slack/email notifications for new contributions (defer to post-GSoC)
- Admin analytics dashboard with charts (defer to post-GSoC)

---

## Risks & mitigations

- **NVD rate limits / downtime**: caching, exponential backoff, CI fixtures.
- **False positives**: conservative heuristics; human verification required before any downstream action.
- **Privacy / exploit leakage**: store only public, non‑sensitive fields; never store exploit payloads or private advisory content.

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

## Technical integration points
- Hook rewards into `website/feed_signals.py` (`giveBacon()`), with an auditable payout ledger model in `website/models.py`.
- Extend `website/challenge_signals.py` for security challenge progress updates.
- Add `SecurityLeaderboardView` in `website/views/leaderboard.py` (or `website/views/user.py` if that’s the current pattern).
- Add API endpoints in `website/api/` or existing API module and route them in `blt/urls.py`.
- Add admin audit views in `website/admin.py` and templates under `website/templates/security/`.

## API design (Project B)
- **Authentication:** Django session + CSRF for web; token-based auth for external consumers.
- **Base path:** `/api/v1/security/leaderboard/` and `/api/v1/security/rewards/`.
- **Pagination:** PageNumberPagination (default 25 per page).
- **Rate limiting:** 100 requests/hour per user (aligned with existing throttling).
- **Filtering:** time range, severity, repo, contributor.
- **Errors:** consistent JSON error shape with `code`, `detail`, and `field_errors`.

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

- Number of verified contributions awarded (pilot target: ≥ 8–12).
- Leaderboard correctly ranks contributors by severity-weighted scores (validated with test fixtures).
- Leaderboard API response time < 500ms for top 100 contributors.
- At least 3 pilot contributors visible on leaderboard with distinct scores.
- Badge issuance counts and distribution.
- Dispute/revocation counts (should be low).

---

## Minimum Viable Scope (if behind schedule)

**Guaranteed minimum deliverables:**
- `VerifiedSecurityContribution` signal/event abstraction ✓
- Reward calculation service with severity-weighted scoring ✓
- BACON distribution hooks (off-chain logging during pilot) ✓
- Badge system: 3 core badges (First Fix, Security Sentinel, CVE Hunter) ✓
- Basic leaderboard view (table with ranking, user, score, contributions) ✓
- Leaderboard API: `GET /api/v1/security/leaderboard/` with pagination ✓
- Challenge tracking: Link contributions to challenges ✓
- Idempotency guarantee (unique constraints + transactional logic) ✓
- Integration tests for full workflow (mock event → reward → leaderboard update) ✓
- Pilot with 8-12 verified contributions across 3+ contributors ✓

**Stretch goals (may be deferred post-GSoC):**
- Advanced badges (10+ badge types with complex criteria)
- Challenge gamification features (time-limited challenges, team challenges)
- Contributor profile pages with full history and badges
- Admin analytics dashboard with fraud detection metrics
- Real-time leaderboard updates via WebSockets (fallback: polling)
- Email notifications for badge awards and leaderboard changes

---

## Risks & mitigations

- **Reward abuse**: monthly caps, reputation gating, and manual steward oversight.
- **Idempotency bugs**: strong DB constraints and transactional award records.
- **BACON economics**: start with simulated or logged payouts during pilot to validate behavior before “real” rewards.

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

## Technical approach (standalone)
- Extend existing Course/Lecture concepts or create a dedicated `education` app.
- Build lab content as structured markdown + metadata with sandboxed exercises.
- Implement auto-quiz grading and an instructor review queue.
- Integrate badge/leaderboard checks for gated access (read-only).

## Milestone outline (standalone)
- **Weeks 0–2:** scaffold + first lab + quiz.
- **Weeks 3–6:** auto-grading + review queue + 3–5 labs.
- **Weeks 7–10:** instructor tooling + badge-based unlocks.
- **Weeks 11–12:** pilot run + documentation + improvements.

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

## Minimum Viable Scope (if behind schedule)

**Guaranteed minimum deliverables:**
- Repository scaffold (blt-education) with Django app structure ✓
- Learning track model (Beginner → Intermediate) ✓
- 4 hands-on labs (XSS, SQLi, CSRF, Path Traversal) ✓
- Sandboxed environment for safe practice (Docker containers or isolated Django views) ✓
- Auto-quiz engine with multiple-choice questions ✓
- Lab completion tracking linked to BLT badges ✓
- Basic student UI (lab list, progress tracker, lab runner) ✓
- Integration with BLT's Badge system for unlocks ✓
- Pilot with 10-15 students ✓

**Stretch goals (may be deferred post-GSoC):**
- 6 labs instead of 4 (add Command Injection, Broken Auth)
- Instructor review queue for manual grading (fallback: auto-grading only)
- Mentor/cohort administration tools
- Advanced learning track (Trusted contributor level)
- Real-world simulation mode with anonymized BLT bug reports
- Community-contributed lab submission workflow

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

## Technical approach (standalone)
- Build an aggregation pipeline to transform BLT data into safe, anonymized metrics.
- Render dashboards with existing chart tooling and exportable reports.
- Create a playbook authoring template with a two-person approval workflow.

## Milestone outline (standalone)
- **Weeks 0–2:** aggregation schema + anonymization rules.
- **Weeks 3–6:** dashboards + report generator MVP.
- **Weeks 7–10:** playbook templates + approval workflow.
- **Weeks 11–12:** pilot report + documentation + refinements.

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

## Minimum Viable Scope (if behind schedule)

**Guaranteed minimum deliverables:**
- Data aggregation service (anonymize BLT security contributions) ✓
- Public-facing dashboard with 3 key metrics (CVE trends, severity distribution, top contributors anonymized) ✓
- 3 remediation playbook templates (SQLi, XSS, Dependency Vulnerabilities) ✓
- Two-person approval workflow for playbook publishing ✓
- Basic report generator (monthly summary in Markdown format) ✓
- Redaction policy documentation ✓
- Privacy review checklist ✓

**Stretch goals (may be deferred post-GSoC):**
- 5 playbook templates instead of 3
- Automated quarterly reports with charts (Chart.js visualizations)
- Case study generator with privacy controls
- Advanced dashboards (time-series analysis, repository comparisons)
- Export functionality (CSV, JSON) for researchers
- Public API for aggregated data access

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

────────────────────────────────────────────────────────────
Alternative (preferred) — Project B + light C (Single 350‑Hour Project)
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

## Minimum Viable Scope (if behind schedule)

**Guaranteed minimum deliverables:**

**From Project B (Core):**
- `VerifiedSecurityContribution` abstraction ✓
- Severity-weighted reward calculation ✓
- BACON distribution hooks (off-chain during pilot) ✓
- 3 core badges (First Fix, Security Sentinel, CVE Hunter) ✓
- Basic leaderboard (table view + API endpoint) ✓
- Challenge tracking ✓
- Idempotency and tests ✓
- Pilot with 8-12 verified contributions ✓

**From light C (Education Bridge):**
- Read-only API: `GET /api/v1/users/{id}/badges` ✓
- Read-only API: `GET /api/v1/users/{id}/security-stats` (returns counts by severity, last_verified_at — no CVE IDs, repo names, or links) ✓
- Badge-gating documentation (how future courses can use these APIs) ✓
- API authentication and rate limiting ✓
- OpenAPI schema documentation ✓

**Stretch goals (may be deferred post-GSoC):**
- Advanced badges (10+ types)
- Challenge gamification features
- Contributor profile pages
- Admin analytics dashboard
- 1-2 sample integration examples showing how a future education platform could consume the APIs

---

## Risks & mitigations

- **Coupling to Project A**:
  - Use mocked verification events if A is not ready.
  - Define GHSC event schema early and code to that contract.
- **Education misuse**:
  - Education endpoints only expose badge/leaderboard data.
  - No raw vulnerability descriptions or exploit content.

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
  - Basic UI shell using BLT's existing UI stack and component patterns.

- **Tiered learning tracks & labs**
  - Learning tracks: **Beginner → Intermediate → Trusted**.
  - Implement 4–6 hands‑on labs:
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

## Minimum Viable Scope (if behind schedule)

**Guaranteed minimum deliverables:**

**From Project C (Education):**
- Repository scaffold (blt-education) ✓
- Learning track model (Beginner → Intermediate) ✓
- 4 hands-on labs (XSS, SQLi, CSRF, Path Traversal) ✓
- Sandboxed environment ✓
- Auto-quiz engine ✓
- Lab completion → badge integration ✓
- Basic student UI ✓
- Pilot with 10-15 students ✓

**From Project D (Knowledge Sharing):**
- Data aggregation service (anonymization) ✓
- Public dashboard with 3 key metrics ✓
- 3 remediation playbook templates ✓
- Two-person approval workflow ✓
- Basic monthly report generator ✓
- Privacy documentation ✓

**Shared Infrastructure:**
- Integrated UI (education + knowledge dashboards) ✓
- Shared governance workflow for content publishing ✓

**Stretch goals (may be deferred post-GSoC):**
- 6 labs instead of 4
- 5 playbooks instead of 3
- Instructor review queue
- Advanced analytics dashboards
- Automated quarterly reports with visualizations
- Community-contributed labs and playbooks submission system

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

## Testing Strategy (All Projects)
- **Unit tests:** target 85%+ coverage using pytest-django where applicable.
- **Integration tests:** critical workflows (e.g., PR merge → GHSC → verify → reward → leaderboard).
- **E2E tests:** key user flows using Playwright (verification dashboard, leaderboards).
- **Fixtures:** reusable fixtures for CVEs, PRs, users, and verification events.
- **CI:** GitHub Actions running tests on every PR.

## Governance & Controls

### Quality control mechanisms
- **Multi-tier review for BACON rewards**
  - Automated checks followed by mentor approval.
  - Build on the existing `BaconSubmission` workflow.
  - Quality metrics and contribution scoring thresholds.
- **Cooldowns and rate limiting**
  - Extend existing 24-hour social connection cooldown pattern.
  - Integrate with Points/Badge reputation systems.
- **Labeling strategy**
  - Distinguish UI/UX bugs from security vulnerabilities.
  - Use severity indicators for security issues.
- **Competitive elements**
  - Reward sustained quality over volume.
  - Weight rewards by verified severity.

### Scope boundaries
- Initial focus on open-source repositories (not closed-source/enterprise).
- No automated exploit disclosure or storage of private advisory content.

### Open questions
- Should UI/UX and security bug discovery be separate workflows or unified?
- What disclosure and contribution rules apply to automated vulnerability submissions?
- What is the right notification strategy and maintainer consent model for proactive scanning?
- What is the minimum viable Buttercup (or equivalent) evaluation required for Project A?
- BACON on-chain distribution depends on mainnet sync completion; during GSoC, log rewards off-chain and switch to on-chain once maintainers confirm readiness.

## Technical clarifications (current assumptions)
- **GHSC vs Issue/Bug models:** GHSC coexists with existing Issue/Bug models and links to them; it does not replace them.
- **BACON mainnet sync:** reward issuance can be logged off-chain during pilot; on-chain payouts follow once mainnet sync is complete.
- **Verification dashboard placement:** new `website/views/security.py` with templates under `website/templates/security/`.
- **Leaderboards:** security leaderboard is a new severity-weighted view extending existing leaderboard patterns; it does not replace global leaderboards.
- **Disputes/clawbacks:** disputed or revoked GHSC entries trigger a reversal record in the payout ledger and a leaderboard recompute.

## Communication & Progress Tracking
- **Weekly sync meetings:** 30–60 minutes with mentor(s) to discuss blockers and next steps.
- **Bi-weekly blog posts:** Public progress updates on BLT blog or GitHub Discussions.
- **Daily async updates:** Brief status in Slack/Discord (what I did, what's next, any blockers).
- **Milestone demos:** Live demos at weeks 4, 7, 10, 12 (midterm checkpoints and final).

## Documentation cross-references
- `docs/features.md` for current capability descriptions.
- `docs/architecture.md` for technical foundation context.
- `BACON/BLT BTC Runes documentation .md` for blockchain reward details.
- `docs/feature-checklist.md` for implementation status tracking.
