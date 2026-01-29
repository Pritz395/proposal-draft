# Project Ideas — Brief Overview

A short reference of BLT GSoC project options. For full technical details, milestones, and scope see [second_review.md](second_review.md).

---

## At a glance

- **Four standalone project options (A–D):** each is a full 350-hour GSoC project on its own.
- **One add-on (“light C”):** not a standalone project. It extends **Project B** with read-only APIs and optional webhooks so future education tools can use badges/leaderboards to gate or unlock content. The **recommended** single 350-hour project is **B + light C** — i.e. Project B plus this add-on.
- **Other combinations:** C and D can be combined into one 350-hour project (education + knowledge sharing). A and B are kept as separate projects.

---

## Purpose

Synthesizes community direction (Discussion #5495) and outlines four GSoC project options (A–D). Each standalone project fits one 350-hour slot. The preferred proposal is **Project B plus the light C add-on** in a single slot.

---

## The four standalone options (A, B, C, D)

### Project A — CVE Detection & Validation Pipeline

**One line:** Opt-in pipeline from scanner/GitHub → NVD validation → GHSC model and verification UI/API.

**Description:** Discovers CVE-related contributions from webhooks and scanner output (e.g. Buttercup), validates against NVD, deduplicates and scores findings, and exposes them via a maintainer verification dashboard and REST API. Post-disclosure only; no raw exploit storage. Foundation for downstream rewards and education.

---

### Project B — Security Contribution Gamification & Recognition

**One line:** Consume verified security contributions to award BACON/badges, reputation tiers, leaderboards, and challenges.

**Description:** Listens for verified GHSC (or equivalent) events and awards rewards idempotently: BACON, badges, reputation tiers (Beginner → Trusted), severity-weighted leaderboards, and security challenges. Includes admin audit and basic fraud controls. Does not do detection or NVD; assumes a feed of verified contributions (real or mocked).

**Add-on: light C (education bridge)**  
Project B can be extended with a **light C** add-on in the same 350-hour slot. Light C is *not* a separate project: it adds read-only APIs and an optional webhook that expose badge/reputation and leaderboard data (no raw CVE or vulnerability details). Future education platforms can use these to unlock courses or show contributor standing. No labs, no curriculum — just the APIs so B’s outputs can drive education tooling. The **recommended** proposal is **B + light C** as one project.

---

### Project C — blt-education Platform (standalone)

**One line:** Tiered learning tracks, hands-on labs, auto-quizzes, and instructor review workflows.

**Description:** Structured security education: Beginner → Intermediate → Trusted tracks, 4–6 labs, auto-grading, instructor review queue, and optional badge-based unlocks. High content and mentoring load; best when education/content capacity exists.

---

### Project D — Knowledge Sharing & Community Impact (standalone)

**One line:** Anonymized aggregation, public dashboards, reports, and remediation playbooks.

**Description:** Pipeline to anonymize and aggregate BLT security data, then publish dashboards, monthly/quarterly reports, and 3–5 remediation playbooks. Includes two-person approval for sensitive content. Depends on having meaningful data to aggregate.

---

## Differentiation (standalone options)

| Project | Focus | Beneficiaries | Dependencies | Risk level |
|---------|--------|---------------|--------------|------------|
| A | Detection + validation | Maintainers, contributors | NVD, scanning | High (false positives) |
| B | Rewards + recognition | Active contributors | Verified signals (or mocks) | Medium (gaming, economics) |
| C | Education platform | New contributors | Content, mentoring | Medium (content burden) |
| D | Knowledge sharing | OSS ecosystem | Aggregated data, governance | Medium (privacy) |

*The recommended single project is **B + light C**: Project B plus the education-bridge add-on described above.*

---

## Recommendation & interest

- **Recommendation:** **Project B + light C** — immediate community value (rewards, leaderboards, challenges) plus a small education bridge so future tools can consume badges/leaderboards. Fits one 350-hour slot; B can run on mocked verification events if Project A is not live.
- **Alternate:** **Project A** if the priority is to build the verification pipeline first.
- **Personal interest:** **Project B + light C** as one 350-hour project: core B (rewards, leaderboards) plus the light C add-on (APIs/webhooks only, no curriculum).

---

## Decision guide

- **Immediate contributor engagement** → **Project B + light C**
- **Strong education/content team** → **Project C + D** (combined into one 350h project)
- **Foundational pipeline first** → **Project A**

---

## Cross-cutting notes

- **Decoupling B from A:** B is designed around a generic “verified security contribution” event; it does not require Project A. Fixtures or a small admin UI can supply events during GSoC; A→B integration is optional later.
- **A + B in one 350-hour slot:** Not recommended; both need focused scope, testing, and pilot time. Treat as two separate projects.
- **C + D combined:** One 350-hour project is possible: education platform (tracks, labs, quizzes, review) plus knowledge-sharing (anonymization, dashboards, playbooks, approval workflow). Shares data and governance concerns.

---

*Full specs, milestones, APIs, and MVP scope are in [second_review.md](second_review.md).*

