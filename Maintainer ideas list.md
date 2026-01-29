# Project Ideas — Brief Overview

A short reference of BLT GSoC project options and their descriptions. For full technical details, milestones, and scope see [second_review.md](second_review.md).

---

## Purpose

Synthesizes community direction (Discussion #5495) and outlines four GSoC project options (A–D) plus a minimal “light C” bridge. Each project fits a single 350-hour slot.

---

## Recommendation & interest

- **Recommendation:** **Project B + light C** — immediate community value (rewards, leaderboards, challenges) plus a small education bridge (read-only APIs for badges/leaderboards). Works with mocked verification events if Project A is not live.
- **Alternate:** **Project A** if the priority is to build the verification pipeline first.
- **Personal interest:** **Project B + light C** as one 350-hour project: core rewards/leaderboards plus a minimal education bridge (APIs/webhooks only, no curriculum).

---

## Project options (brief)

### Project A — CVE Detection & Validation Pipeline

**One line:** Opt-in pipeline from scanner/GitHub → NVD validation → GHSC model and verification UI/API.

**Description:** Discovers CVE-related contributions from webhooks and scanner output (e.g. Buttercup), validates against NVD, deduplicates and scores findings, and exposes them via a maintainer verification dashboard and REST API. Post-disclosure only; no raw exploit storage. Foundation for downstream rewards and education.

---

### Project B — Security Contribution Gamification & Recognition

**One line:** Consume verified security contributions to award BACON/badges, reputation tiers, leaderboards, and challenges.

**Description:** Listens for verified GHSC (or equivalent) events and awards rewards idempotently: BACON, badges, reputation tiers (Beginner → Trusted), severity-weighted leaderboards, and security challenges. Includes admin audit and basic fraud controls. Does not do detection or NVD; assumes a feed of verified contributions (real or mocked).

---

### Project C — blt-education Platform (standalone)

**One line:** Tiered learning tracks, hands-on labs, auto-quizzes, and instructor review workflows.

**Description:** Structured security education: Beginner → Intermediate → Trusted tracks, 4–6 labs, auto-grading, instructor review queue, and optional badge-based unlocks. High content and mentoring load; best when education/content capacity exists.

---

### Project D — Knowledge Sharing & Community Impact (standalone)

**One line:** Anonymized aggregation, public dashboards, reports, and remediation playbooks.

**Description:** Pipeline to anonymize and aggregate BLT security data, then publish dashboards, monthly/quarterly reports, and 3–5 remediation playbooks. Includes two-person approval for sensitive content. Depends on having meaningful data to aggregate.

---

### light C — Education bridge (not standalone)

**One line:** Read-only APIs and optional webhooks for badges and leaderboards so education tools can gate/unlock content.

**Description:** Minimal add-on (no labs or curriculum). Exposes badge/reputation and leaderboard data via APIs and optionally a webhook on badge/tier changes. Education systems can use this to unlock courses. No raw vulnerability or CVE details; aggregate/identity-level only. Often combined with **Project B** in a single 350-hour project.

---

## Differentiation

| Project   | Focus                | Beneficiaries        | Dependencies              | Risk level        |
|----------|----------------------|----------------------|---------------------------|-------------------|
| A        | Detection + validation | Maintainers, contributors | NVD, scanning         | High (false positives) |
| B        | Rewards + recognition | Active contributors  | Verified signals (or mocks) | Medium (gaming, economics) |
| C        | Education platform   | New contributors     | Content, mentoring       | Medium (content burden) |
| D        | Knowledge sharing    | OSS ecosystem        | Aggregated data, governance | Medium (privacy) |
| light C  | Bridge only          | Education tooling    | B (badges/leaderboards)   | Low               |

---

## Decision guide

- **Immediate contributor engagement** → **Project B + light C**
- **Strong education/content team** → **Project C + D** (combined)
- **Foundational pipeline first** → **Project A**

---

## Combined & cross-cutting notes

- **B + light C (preferred single project):** Full B (rewards, badges, leaderboards, challenges, audit) plus light C (read-only badge/leaderboard APIs and optional webhook). Single 350-hour scope; B can run on mocked verification events.
- **Decoupling B from A:** B is designed around a generic “verified security contribution” event; it does not require Project A. Fixtures or a small admin UI can supply events during GSoC; A→B integration is optional later.
- **A + B in one 350-hour slot:** Not recommended; both need focused scope, testing, and pilot time. Treat as two separate projects.
- **C + D combined:** One 350-hour project possible: education platform (tracks, labs, quizzes, review) plus knowledge-sharing (anonymization, dashboards, playbooks, approval workflow). Shares data and governance concerns.

---

*Full specs, milestones, APIs, and MVP scope are in [second_review.md](second_review.md).*
