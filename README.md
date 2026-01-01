# GitHub Security Contribution Gamification & Recognition Platform - GSoC Proposal

Building on: BLT's existing GitHubIssue/GitHubReview/webhook infrastructure, CVE fields, BACON, badges, and daily challenges work (PR #5245).

**Goal:** Track security-focused GitHub PRs/reviews/issues (especially post-disclosure CVE fixes), link them to existing CVE data via NVD API validation, reward contributors with BACON/badges after maintainer verification, and surface impact-based leaderboards, dashboards, and challenges around security work.

**Post-Disclosure Scope:** This system tracks CVE contributions **after public disclosure** (respects responsible disclosure practices). Cannot track private/embargoed security work due to GitHub API limitations (private advisories deprecated May 2024) and ethical disclosure practices. Most CVE work eventually becomes public, making post-disclosure tracking practical and secure.

---

## System Architecture

### Architecture Diagram

```mermaid
graph TB
    subgraph "GitHub Integration Layer"
        GH[GitHub Webhook]
        GHPR[Pull Request Events]
        GHREV[Review Events]
    end

    subgraph "BLT Webhook Handler (Existing)"
        WEBHOOK[github_webhook view]
        PRHANDLER[handle_pull_request_event]
        REVHANDLER[handle_review_event]
    end

    subgraph "CVE Detection & Validation"
        CVEDETECT[CVE Pattern Matcher]
        NVDAPI[NVD API Client<br/>nvdlib]
        CVESCORE[CVE Severity Calculator]
    end

    subgraph "Database Models (Existing + New)"
        GHI[GitHubIssue<br/>Existing]
        GHR[GitHubReview<br/>Existing]
        ISSUE[Issue<br/>cve_id, cve_score<br/>Existing]
        GHSC[GitHubSecurityContribution<br/>NEW MODEL]
        USER[User/UserProfile<br/>Existing]
        CONTRIB[Contributor<br/>Existing]
        REPO[Repo<br/>Existing]
    end

    subgraph "Verification Workflow"
        VERIFYVIEW[SecurityContributionVerificationView<br/>NEW VIEW]
        VERIFIER[Maintainer/Verifier<br/>can_verify_security_contributions]
        VERIFYSTATUS[verification_status<br/>pending/verified/rejected/disputed]
        RATELIMIT[Rate Limiting<br/>5/month per contributor]
    end

    subgraph "Reward System (Existing)"
        SIGNAL[post_save signal<br/>on verification]
        GIVEBACON[giveBacon function<br/>feed_signals.py]
        BACON[BaconEarning<br/>Existing]
        BADGE[Badge/UserBadge<br/>Existing]
    end

    subgraph "Leaderboard System (Existing + Extension)"
        LBASE[LeaderboardBase<br/>Existing]
        SECLB[SecurityLeaderboardView<br/>EXTENDS LeaderboardBase]
        GLOBLB[GlobalLeaderboardView<br/>Existing]
    end

    subgraph "Challenge System (Existing + Extension)"
        CHALLENGE[Challenge Model<br/>Existing]
        CHALSIG[challenge_signals.py<br/>update_challenge_progress]
        SECCHAL[Security Daily Challenges<br/>EXTENDS Challenge]
    end

    subgraph "Analytics & Security"
        RATELIMITMW[Rate Limiting Middleware<br/>Existing]
        SPAMDETECT[Spam Detection<br/>Duplicate Check<br/>Reputation Tracking]
        ANALYTICS[Security Analytics Dashboard<br/>Chart.js Visualizations<br/>NEW VIEW]
    end

    subgraph "UI Components"
        SECDASH[Security Dashboard<br/>NEW TEMPLATE]
        VERIFYDASH[Verification Dashboard<br/>NEW TEMPLATE]
        LEADERBOARDUI[Leaderboard UI<br/>Existing]
    end

    %% Flow connections
    GH -->|POST webhook| WEBHOOK
    WEBHOOK -->|routes| PRHANDLER
    WEBHOOK -->|routes| REVHANDLER

    PRHANDLER -->|creates/updates| GHI
    REVHANDLER -->|creates/updates| GHR

    PRHANDLER -->|detects CVE in PR| CVEDETECT
    CVEDETECT -->|extracts CVE ID| NVDAPI
    NVDAPI -->|validates & fetches| CVESCORE
    CVESCORE -->|creates| GHSC

    GHSC -->|references| GHI
    GHSC -->|references| GHR
    GHSC -->|references| USER
    GHSC -->|references| CONTRIB
    GHSC -->|references| REPO
    GHSC -->|cve_id links to| ISSUE

    GHSC -->|status: pending| VERIFYVIEW
    VERIFIER -->|reviews in| VERIFYVIEW
    VERIFYVIEW -->|updates| VERIFYSTATUS

    VERIFYSTATUS -->|verified| SIGNAL
    SIGNAL -->|triggers| GIVEBACON
    GIVEBACON -->|awards| BACON
    SIGNAL -->|awards| BADGE

    GHSC -->|aggregated| SECLB
    SECLB -->|extends| LBASE
    LBASE -->|used by| GLOBLB

    GHSC -->|tracks| SECCHAL
    SECCHAL -->|extends| CHALLENGE
    CHALLENGE -->|updates via| CHALSIG

    GHSC -->|monitored by| RATELIMIT
    GHSC -->|analyzed by| SPAMDETECT
    GHSC -->|displayed in| ANALYTICS

    VERIFYVIEW -->|renders| VERIFYDASH
    ANALYTICS -->|renders| SECDASH
    SECLB -->|renders| LEADERBOARDUI

    style SECLB fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff

    style GHSC fill:#e74c3c,stroke:#c0392b,stroke-width:3px,color:#fff
    style VERIFYVIEW fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
    style NVDAPI fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
    style SECCHAL fill:#e74c3c,stroke:#c0392b,stroke-width:2px,color:#fff
```

### Database Entity Relationship Diagram

```mermaid
erDiagram
    User ||--o{ UserProfile : has
    User ||--o{ UserBadge : "earns"
    User ||--o{ BaconEarning : "accumulates"
    User ||--o{ GitHubSecurityContribution : "creates"

    UserProfile ||--o{ GitHubIssue : "creates"
    UserProfile ||--o{ GitHubReview : "reviews"
    UserProfile ||--o{ Issue : "reports"

    Contributor ||--o{ GitHubIssue : "contributes"
    Contributor ||--o{ GitHubReview : "reviews"
    Contributor ||--o{ GitHubSecurityContribution : "contributes"

    Repo ||--o{ GitHubIssue : "contains"
    Repo ||--o{ GitHubSecurityContribution : "tracks"

    GitHubIssue ||--o{ GitHubReview : "has"
    GitHubIssue ||--o{ GitHubSecurityContribution : "references"

    Issue ||--o{ GitHubSecurityContribution : "linked via CVE"

    Badge ||--o{ UserBadge : "awarded as"

    Challenge ||--o{ GitHubSecurityContribution : "tracks"

    User {
        int id PK
        string username
        string email
    }

    UserProfile {
        int id PK
        int user_id FK
        string github_url
        boolean is_verifier
    }

    Contributor {
        int id PK
        int github_id UK
        string name
        string github_url
    }

    Repo {
        int id PK
        string repo_url UK
        string name
    }

    GitHubIssue {
        bigint id PK
        bigint issue_id
        int repo_id FK
        int user_profile_id FK
        int contributor_id FK
        string type
        boolean is_merged
        datetime merged_at
    }

    GitHubReview {
        bigint id PK
        bigint review_id UK
        bigint pull_request_id FK
        int reviewer_id FK
        int reviewer_contributor_id FK
        string state
    }

    Issue {
        int id PK
        int user_id FK
        string cve_id
        decimal cve_score
    }

    GitHubSecurityContribution {
        int id PK
        bigint github_issue_id FK
        bigint github_review_id FK
        int contributor_id FK
        int repo_id FK
        string cve_id
        decimal cve_score
        string severity
        string verification_status
        int verifier_id FK
        datetime verified_at
        datetime created_at
    }

    Badge {
        int id PK
        string title
        string type
    }

    UserBadge {
        int id PK
        int user_id FK
        int badge_id FK
        int awarded_by_id FK
        datetime awarded_at
    }

    BaconEarning {
        int id PK
        int user_id FK
        int tokens_earned
    }

    Challenge {
        int id PK
        string title
        string challenge_type
        int bacon_reward
    }
```

**Note on Field Types:**
- `Contributor.github_id`: Currently `IntegerField` in codebase. Consider migrating to `BigIntegerField` for GitHub IDs > 2^31 (future-proofing).
- `Issue.cve_score`: Currently `max_digits=2` in codebase. Requires migration to `max_digits=3` to support CVSS scores up to 10.0 (see Database Changes section below).

---

## Complete Workflow Sequence

### Main Flow: PR Merge to BACON Award

```mermaid
sequenceDiagram
    participant GH as GitHub
    participant WH as Webhook Handler<br/>(github_webhook)
    participant PRH as PR Event Handler<br/>(handle_pull_request_event)
    participant CVED as CVE Detector<br/>(NEW)
    participant NVD as NVD API<br/>(nvdlib)
    participant DB as Database<br/>(GitHubSecurityContribution)
    participant VER as Verification View<br/>(SecurityContributionVerificationView)
    participant MAINT as Maintainer<br/>(is_verifier=True)
    participant SIG as Django Signal<br/>(post_save)
    participant BACON as giveBacon<br/>(feed_signals.py)
    participant BADGE as Badge System<br/>(Existing)
    participant CHAL as Challenge System<br/>(challenge_signals.py)
    participant LB as Leaderboard<br/>(SecurityLeaderboardView)

    Note over GH,MAINT: Phase 1: PR Merge Detection
    GH->>WH: POST /github-webhook/<br/>pull_request event<br/>(action=closed, merged=true)
    WH->>WH: Validate HMAC signature
    WH->>PRH: Route to handle_pull_request_event()

    Note over PRH,DB: Phase 2: CVE Detection & Validation
    PRH->>PRH: Extract PR data<br/>(title, body, merged_at)
    PRH->>PRH: Update/create GitHubIssue record
    PRH->>CVED: Check for CVE pattern<br/>(CVE-YYYY-NNNNN)

    alt CVE Pattern Found
        CVED->>CVED: Extract CVE ID from PR text
        CVED->>NVD: GET /rest/json/cves/2.0?cveId={cve_id}
        NVD-->>CVED: CVE data + CVSS score

        CVED->>CVED: Calculate severity<br/>(Critical/High/Medium/Low)
        CVED->>DB: Create GitHubSecurityContribution<br/>status='pending'<br/>severity={calculated}<br/>cve_score={from NVD}

        Note over DB: Links to:<br/>- GitHubIssue<br/>- Contributor<br/>- Repo
    else No CVE Found
        PRH->>PRH: Continue normal PR processing
    end

    Note over VER,MAINT: Phase 3: Maintainer Verification
    DB->>VER: Query pending contributions<br/>(verification_status='pending')
    VER->>MAINT: Display verification dashboard<br/>(SecurityContributionVerificationView)

    MAINT->>VER: Review contribution details<br/>(CVE ID, PR link, severity)
    MAINT->>VER: Click "Verify" or "Reject"

    alt Verified
        VER->>DB: Update GitHubSecurityContribution<br/>verification_status='verified'<br/>verifier_id={maintainer}<br/>verified_at={now}

        Note over SIG,BADGE: Phase 4: Reward Distribution
        DB->>SIG: post_save signal triggered<br/>(verification_status='verified')
        SIG->>BACON: Calculate BACON reward<br/>(Critical=100, High=75,<br/>Medium=50, Low=25)
        BACON->>BACON: Update BaconEarning<br/>tokens_earned += reward_amount

        SIG->>BADGE: Check badge criteria<br/>(e.g., "First CVE Fix",<br/>"Critical Security Expert")
        alt Badge Criteria Met
            BADGE->>BADGE: Create UserBadge record
        end

        Note over CHAL,LB: Phase 5: Challenge & Leaderboard Updates
        SIG->>CHAL: update_challenge_progress()<br/>challenge_title="Fix 5 CVEs"<br/>model=GitHubSecurityContribution
        CHAL->>CHAL: Check if challenge completed
        alt Challenge Completed
            CHAL->>BACON: Award challenge BACON reward
        end

        SIG->>LB: Update SecurityLeaderboardView<br/>Aggregate by contributor
        LB->>LB: Recalculate rankings

        VER-->>MAINT: Success: Contribution verified<br/>BACON awarded
    else Rejected
        VER->>DB: Update GitHubSecurityContribution<br/>verification_status='rejected'<br/>verifier_id={maintainer}<br/>verified_at={now}
        VER-->>MAINT: Contribution rejected<br/>(no rewards)
    end

    Note over GH,LB: End: Contribution tracked in<br/>leaderboards, challenges, and user profile
```

---

## Integration with Existing BLT Infrastructure

### Key Integration Points

1. **GitHub Webhook System** (`website/views/user.py`)

   - Extends `handle_pull_request_event()` to detect CVE references
   - Extends `handle_review_event()` for review-based contributions
   - No changes to webhook routing or signature validation

2. **Database Models**

   - **New:** `GitHubSecurityContribution` model
   - **Uses:** Existing `GitHubIssue`, `GitHubReview`, `Contributor`, `Repo`, `UserProfile`
   - **Links to:** Existing `Issue` model via `cve_id` field

3. **Reward System** (`website/feed_signals.py`)

   - Uses existing `giveBacon()` function
   - Integrates with existing `BaconEarning` model
   - Extends badge system via `Badge`/`UserBadge` models

4. **Leaderboard System** (`website/views/user.py`)

   - Extends `LeaderboardBase` class (no modifications to base)
   - New `SecurityLeaderboardView` inherits all base functionality
   - Integrates with existing `GlobalLeaderboardView`

5. **Challenge System** (`website/challenge_signals.py`)

   - Uses existing `update_challenge_progress()` function
   - Extends `Challenge` model (no schema changes)
   - Integrates with existing challenge completion flow

6. **Verification System**
   - Uses existing `UserProfile.is_verifier` field
   - New view follows existing BLT view patterns
   - No changes to user permission system

---

## Database Changes Required

### Migration 1: Fix Issue.cve_score Field Constraint

**Current:** `Issue.cve_score = models.DecimalField(max_digits=2, decimal_places=1)`  
**Problem:** Max value is 9.9, but CVSS scores go up to 10.0  
**Fix:** Change to `max_digits=3, decimal_places=1` to support scores 0.0-10.0

```python
# Migration needed in website/migrations/
class Migration(migrations.Migration):
    operations = [
        migrations.AlterField(
            model_name='issue',
            name='cve_score',
            field=models.DecimalField(max_digits=3, decimal_places=1, blank=True, null=True),
        ),
    ]
```

### Migration 2: Contributor.github_id Field Type (Optional)

**Current:** `Contributor.github_id = models.IntegerField()`  
**Note:** GitHub user IDs can exceed 32-bit integer limits (2^31). Consider migrating to `BigIntegerField` if needed for future-proofing, though current IntegerField works for most cases.

---

## Files to Create/Modify

### New Files

1. `website/models.py` - Add `GitHubSecurityContribution` model
2. `website/security_signals.py` - Signal handlers for rewards
3. `website/services/cve_service.py` - NVD API integration
4. `website/views/security.py` - Verification dashboard view
5. `website/templates/security/verification_dashboard.html` - Maintainer UI
6. `website/templates/leaderboard_security.html` - Security leaderboard UI
7. `website/templates/security/dashboard.html` - User-facing dashboard
8. `website/static/js/security-dashboard.js` - Client-side interactions
9. `website/migrations/XXXX_create_github_security_contribution.py` - Database migration

### Modified Files

1. `website/views/user.py` - Extend webhook handlers, add SecurityLeaderboardView
2. `website/challenge_signals.py` - Add security challenge handlers
3. `blt/urls.py` - Add security routes
4. `website/admin.py` - Register new model
5. `blt/settings.py` - Add NVD API config

### Dependencies

```toml
[tool.poetry.dependencies]
nvdlib = "^0.3.0"  # NVD API client
```

---

## Implementation Timeline (~350-370h total)

### Milestone 1: CVE Detection & Tracking 

- Extract CVE IDs from merged PR/issue content using `CVE-\d{4}-\d{4,7}` regex pattern (to be created)
- Create CVE regex pattern utility function
- Link to `Issue.cve_id`, store in new `GitHubSecurityContribution` model
- Wire into `github_webhook` handlers
- Update `handle_pull_request_event()`, `handle_review_event()`, `handle_issue_event()` to detect CVE references
- **Post-disclosure scope:** Track CVE contributions after public disclosure (respects responsible disclosure practices)

### Milestone 2: NVD API Validation & Pre-Screening

- Integrate `nvdlib` (NIST NVD API wrapper) to validate CVE IDs exist
- Fetch severity scores (CRITICAL/HIGH/MEDIUM/LOW) and descriptions
- Pre-screen contributions to block:
  - Fake CVE IDs
  - Documentation-only PRs claiming CVE credit
  - Duplicate submissions
  - Contributors with spam history
- Rate limiting: Max 5 CVE contribution claims per contributor per month (Django cache)

### Milestone 3: Maintainer Verification Workflow 

- Build `SecurityContributionVerificationView` admin dashboard for maintainers
- New model fields: `verification_status`, `verified_by`, `verification_notes`
- Permissions: Add `can_verify_security_contributions` permission
- Dashboard shows: CVE details from NVD, PR diff, contributor history
- Actions: Verify, Reject, Request More Info, Dispute

### Milestone 4: Reward Distribution 

- Use `giveBacon()` from `feed_signals.py` so verified CVE-fix PRs auto-earn BACON
- Scaled by severity: Critical=100, High=75, Medium=50, Low=25
- Security badges via existing `Badge`/`UserBadge` models
- Reward gating: BACON only awarded after maintainer verification (via Django signal)
- Tracks `bacon_awarded` and `bacon_amount` fields

### Milestone 5: Security Leaderboards 

- Build leaderboards extending `LeaderboardBase`
- Rank by impact (`cve_score × contribution count`)
- Separate views for PRs, reviews, and security issues
- Time-period filtering (monthly, yearly, all-time)
- Contributor reputation metrics

### Milestone 6: Security Daily Challenges

- Add GitHub security challenge types to `Challenge.CHALLENGE_TYPE_CHOICES` (depends on PR #5245 being merged first)
- Extend `update_challenge_progress()` to validate via `GitHubSecurityContribution`
- Examples: "Fix a MEDIUM+ CVE this week", "Review 3 security PRs"

### Milestone 7: Security Dashboard UI 

- Create accessible, dark-mode `SecurityContributionDashboardView`
- Shows contributions, badges, verification status, timelines, impact metrics
- Reuse ARIA/dark-mode patterns from PRs #4937 and #5169
- Charts for CVE trends (Chart.js), severity distribution, top contributors

### Milestone 8: Analytics, Tests & Docs 

- `SecurityAnalyticsView` with Django ORM aggregations for CVE fixes over time
- Top contributors/orgs analytics
- End-to-end tests extending `test_github_webhook.py`
- Fraud scenario tests: Fake CVEs, duplicate submissions, trivial PRs, rate limit violations
- Maintainer verification guide
- Contributor CVE claiming guide
- Security audit of verification logic

---

## Security & Fraud Prevention

1. **Rate Limiting**

   - Uses existing middleware (`blt/middleware/throttling.py`)
   - Prevents spam contribution creation

2. **Verification Requirement**

   - All contributions require maintainer verification
   - No automatic rewards without verification

3. **CVE Validation**

   - NVD API validates CVE IDs
   - Invalid CVEs marked as 'Unknown' severity
   - Maintainer can override during verification

4. **Audit Trail**
   - Tracks verifier and verification timestamp
   - Rejection reasons stored
   - All changes logged

---

## Success Criteria

1. CVE detection works for merged PRs
2. NVD API integration validates CVEs
3. Maintainer verification workflow functional
4. BACON rewards awarded correctly (Critical=100, High=75, Medium=50, Low=25)
5. Badges awarded for milestones
6. Security leaderboard displays correctly
7. Challenges track security contributions
8. Fraud prevention measures effective
9. Performance acceptable (<500ms for verification)
10. Documentation complete

---

## Scope & Limitations

### What This Tracks

- Publicly disclosed CVE fixes (post-disclosure only)
- Merged PRs with CVE references
- Published GitHub Security Advisories
- Security reviews and security-labeled issues
- Security-labeled PRs (if GitHub provides labels)

### What This Doesn't Track 

-  Private/embargoed CVE work (respects responsible disclosure)
-  Unpublished security advisories
-  Pre-disclosure contributions

**Why Post-Disclosure?**

- **Ethical:** Respects responsible disclosure practices
- **Secure:** No premature vulnerability leaks
- **Practical:** Most CVE work eventually becomes public
- **API Limitation:** GitHub deprecated private advisory access (May 2024)

## Data Flow Summary

```
GitHub PR Merge (with CVE reference)
    ↓
Webhook Handler (detects CVE pattern)
    ↓
Pre-Screening:
  - Check rate limit (5/month)
  - Check duplicate submission
  - Check spam history
    ↓
CVE Service (validates via NVD API)
    ↓
Create GitHubSecurityContribution (status='pending')
    ↓
Verification Dashboard (maintainer reviews)
  - Shows CVE details from NVD
  - Shows PR diff
  - Shows contributor history
    ↓
Update status to 'verified' (or 'rejected'/'disputed')
    ↓
Django Signal triggers (only if verified)
    ↓
giveBacon() awards tokens (Critical=100, High=75, Medium=50, Low=25)
    ↓
Badge system awards badges
    ↓
Challenge system updates progress
    ↓
Leaderboard updates rankings
```

---

## UI/UX Considerations

- Uses Tailwind CSS (existing BLT standard)
- Brand color: `#e74c3c` (red)
- Responsive design with dark-mode support
- Real-time updates via AJAX
- Clear verification workflow
- User-friendly error messages
- Chart.js visualizations for CVE trends, severity distribution
- Accessible design (ARIA patterns from PRs #4937 and #5169)
- Security dashboard shows:
  - Contributions timeline
  - Badges earned
  - Verification status
  - Impact metrics
  - CVE trends charts
  - Top contributors visualization

---

## Bonus: MY-GSOC-TOOL Integration

Since we're tracking security contribution data, it could connect to MY-GSOC-TOOL, so students could showcase security PRs in GSoC portfolios. Not part of core scope, but easy to add a REST API endpoint later.

## Related Work

- **PR #5057:** CVE Search, Filtering, Caching, Autocomplete (Open) 
- **PR #5245:** Daily Challenges with 24-Hour Reset (Open) 
- **PR #5371:** Security Headers & CSRF (Open) 
- **PR #5351:** Session Security, Rate Limiting (Open) 

**Note:** This proposal builds entirely on top of existing BLT infrastructure. No existing features are replaced or removed. All new functionality integrates seamlessly with current systems including GitHub webhooks, BACON rewards, badges, challenges, and leaderboards.
