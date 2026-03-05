# Community Staff Onboarding System — Technical Specification

> **Version:** 1.0
> **Date:** March 1, 2026
> **Status:** RFC - Request for Comments
> **Framework:** [Carbon](https://github.com/buape/carbon) (Discord bot framework by Buape)
> **Database:** SQLite (file-based, simple deployment)
> **Documentation:** [openclaw/community](https://github.com/openclaw/community) (primary docs repo)

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [Roles & Permissions](#3-roles--permissions)
4. [Workflow Stages](#4-workflow-stages)
5. [Bot Commands](#5-bot-commands)
6. [Promotion Pipeline](#6-promotion-pipeline)
7. [Notification System](#7-notification-system)
8. [Data Model & Retention](#8-data-model--retention)
9. [Edge Cases & Failure Modes](#9-edge-cases--failure-modes)
10. [Guidelines & Documentation](#10-guidelines--documentation)
11. [Team-Specific Extensions](#11-team-specific-extensions)

---

## 1. Overview

A semi-automated community staff onboarding pipeline for a Discord community of 50,000+ members. The system manages the full lifecycle from application → trial → promotion (or permanent decline) with bot-driven logistics and human decision-making at key checkpoints.

This specification defines the **general onboarding framework** applicable to all OpenClaw community staff teams. Team-specific extensions (e.g., voice-specific features) are documented in separate RFCs.

### Organizational Context

OpenClaw uses a four-team moderation structure:

- **Discord Moderators** (text channels, rules enforcement)
- **VC Moderators** (voice channels)
- **Helpers** (community support, questions)
- **Configurators** (bot & server config management)

All team leads report to Shadow (Administrator). Each team has a dedicated Team Lead who coordinates team activities and manages the onboarding pipeline for their team. All moderators receive the **Community Staff** umbrella role (visible in sidebar, no inherent permissions).

### Entry Point

The **only** way to become a community staff member is by emailing **shadow@openclaw.ai**. Shadow acts as the initial funnel, has a conversation with the candidate, and directs them to the appropriate Team Lead if suitable.

### Design Principles

- **Semi-automated:** Bot handles logistics, tracking, and notifications. All team members vote on promotions, with final sign-off from Team Lead.
- **Team-specific trials:** Each team defines their own trial requirements and success criteria. The bot provides the framework; teams implement their specific evaluation methods.
- **Second chance:** Declined applicants can reapply after a configurable cooldown period (default: 7 days). This ensures fairness while maintaining accountability.
- **Full audit trail:** Every action, decision, and data point is permanently retained.
- **Observer-first:** Trial members observe and learn — they have limited or no moderation powers during trial.
- **Privacy-first for trials:** Trial members remain private (no public visibility) until promotion is confirmed, to avoid embarrassment if they back out.

### Expected Volume

- **10–40 applicants per month** across all teams (steady flow)
- **Pipeline duration:** ~1–2 weeks per applicant (review + trial period)

---

## 2. Architecture

### Framework

**The bot MUST be implemented using the [Carbon framework](https://github.com/buape/carbon)** by Buape. Carbon provides a structured approach to building Discord bots with built-in support for commands, components, and interactions.

### Bot Ecosystem Positioning

This is a **standalone Carbon bot** within OpenClaw's broader bot ecosystem:

- **Barnacle Bot** (2-in-1: Sapphire + Carbon) — multi-purpose server management
- **Audrey** — voice channel automation
- **Krill** — support ticket system
- **Kiai** — XP and leveling system
- **This bot** (Community Staff Onboarding) — dedicated community staff pipeline management across all teams

This bot operates as a separate bot account and codebase, managing onboarding for all four community staff teams.

### High-Level Components

```
┌──────────────────────────────────────────────────────────────┐
│                      Carbon Bot                              │
├──────────────┬──────────────┬──────────────┬─────────────────┤
│  Application │  Onboarding  │   Tracking   │   Promotion     │
│   Module     │   Module     │   Module     │   Module        │
├──────────────┴──────────────┴──────────────┴─────────────────┤
│               Team Extension Module (Pluggable)               │
│           Handles team-specific trial requirements            │
├──────────────────────────────────────────────────────────────┤
│                     Data Layer (Database)                     │
│              Full audit trail, permanent retention            │
└──────────────────────────────────────────────────────────────┘
```

### Required Infrastructure

- **Database:** SQLite for persistent storage of applications, trials, votes, and audit logs
- **Scheduler/Cron:** For deadline checks, trial monitoring, and automated stage transitions
- **Discord Resources:** Roles, private channels, DM capabilities

### Team Extension System

The bot is designed to be extensible. Each team can provide:
- Custom trial evaluation criteria
- Team-specific commands
- Custom tracking metrics
- Additional data fields

Extensions are documented in separate RFCs (e.g., `rfc/voice/voice-001-extensions.md`).

---

## 3. Roles & Permissions

### Discord Roles

| Role | Purpose | Permissions | Granted By |
|------|---------|-------------|------------|
| **Trial Moderator** (org-wide) | Provisional role for people exploring which team to join | Access to one social channel only. No server permissions. | Manual assignment by Shadow |
| **Trial [Team]** (team-specific) | Assigned during team-specific trial period | View team's private guidelines channel. Limited/no moderation permissions. Team-specific trial access. | Bot, via `/onboard-start` |
| **[Team] Staff** (full role) | Assigned upon successful promotion | Team-specific moderation permissions. Can vote on promotions for their team. Also receives Community Staff umbrella role. | Bot, via promotion |
| **Community Staff** (umbrella) | Visible sidebar role for all moderators across all 4 teams | No inherent permissions (permissions granted via team-specific roles). | Bot, on promotion to any team |
| **Team Lead** | Designated lead for each team | All team staff permissions + final approval authority on promotions and ability to manage the onboarding pipeline for their team. Reports to Shadow. | Manual assignment |

**Examples of team-specific roles:**
- Trial Discord Mod → Discord Mod
- Trial VC Mod → VC Mod
- Trial Helper → Helper
- Trial Configurator → Configurator

### Permission Boundaries

- **Trial members** have limited or no moderation powers. They are primarily observers/learners.
- **Full team members** have team-specific moderation permissions. Can participate in promotion votes for their team.
- **Team Leads** provide final approval on all promotions after team votes. Coordinate team activities.

### Security Requirements

- **Discord 2FA:** All community staff (trial and full) must have Discord 2FA enabled.
- **GitHub 2FA:** Required if accessing the openclaw/community documentation repository.

---

## 4. Workflow Stages

```mermaid
flowchart TD
    Start([User emails shadow@openclaw.ai]) --> Shadow{Shadow reviews<br/>candidate}
    Shadow -->|Not appropriate| Decline1[Shadow declines or<br/>suggests different path]
    Shadow -->|Appropriate| TeamLead[Shadow directs to Team Lead]

    TeamLead --> OnboardCmd[Team Lead runs<br/>/onboard-start @user --team]
    OnboardCmd --> InServer{User in<br/>Discord server?}
    InServer -->|No| Error1[Error: User must be in server]
    InServer -->|Yes| DMForm[Bot DMs application form modal]

    DMForm --> UserFills[User completes form:<br/>timezone, availability,<br/>motivation, team questions]
    UserFills --> StoreApp[Bot stores application<br/>in database]

    StoreApp --> AutoCheck{Auto-reject<br/>criteria check}
    AutoCheck -->|Failed:<br/>Server tenure < 14 days OR<br/>Has bans/warnings OR<br/>Already has pending app OR<br/>In cooldown period| AutoReject[Bot DMs: auto-rejected<br/>Can reapply after cooldown<br/>Bot notifies initiator]
    AutoCheck -->|Passed| PostReview[Posted to team channel<br/>for review]

    PostReview --> TeamReview{Team reviews<br/>application<br/>2 full members OR<br/>1 Team Lead}
    TeamReview -->|Any member denies| Deny1[Status: APPLICATION_DENIED<br/>Bot DMs: application denied<br/>Can reapply after cooldown]
    TeamReview -->|Approved| GrantRole[Status: TRIAL_ACTIVE<br/>Bot grants Trial Team role]

    GrantRole --> UnlockChannel[Bot unlocks team's private<br/>guidelines channel read-only]
    UnlockChannel --> DMOnboard[Bot DMs applicant:<br/>- Confirmation<br/>- Guidelines link<br/>- Trial expectations<br/>- Failure conditions]
    DMOnboard --> TrialStart[Trial member begins<br/>team-specific trial activities]

    TrialStart --> TrialActive{Trial period}
    TrialActive -->|During trial:<br/>- Follows team activities<br/>- Bot tracks metrics<br/>- Team observes| TrialActive
    TrialActive -->|Failed requirements| TrialFail[Bot removes Trial Team role<br/>Bot DMs: trial failed<br/>Mark as TRIAL_FAILED<br/>Can reapply after cooldown]
    TrialActive -->|All requirements<br/>completed| TrialComplete[Bot confirms completion<br/>in team channel]

    TrialComplete --> OpenVote[Status: AWAITING_TEAM_VOTE<br/>Bot opens promotion vote<br/>with approve/deny buttons<br/>48-hour window]
    OpenVote --> VoteWindow[Team members vote]

    VoteWindow --> Majority{Simple majority<br/>>50% reached?}
    Majority -->|No| VoteFail[Status: VOTE_FAILED<br/>Bot removes Trial Team role<br/>Bot DMs: declined<br/>Can reapply after cooldown]
    Majority -->|Yes| AwaitingLead[Status: AWAITING_LEAD_APPROVAL<br/>Notify Team Lead<br/>Start 7-day timeout]

    AwaitingLead --> LeadReview{Team Lead<br/>responds?}
    LeadReview -->|/deny within 7 days| LeadDeny[Status: DENIED_BY_LEAD<br/>Record Team Lead ID<br/>Bot removes Trial Team role<br/>Bot DMs: declined by Lead<br/>Can reapply after cooldown]
    LeadReview -->|/promote within 7 days| Promote[Status: PROMOTED_BY_LEAD<br/>Record Team Lead ID<br/>Bot promotes to full Team Staff<br/>Bot grants Community Staff role<br/>Bot removes Trial Team role]
    LeadReview -->|No response after 7 days| AutoPromote[Status: PROMOTED_BY_LEAD_INACTION<br/>Record Team Lead ID (at timeout)<br/>Bot promotes to full Team Staff<br/>Bot grants Community Staff role<br/>Bot removes Trial Team role<br/>Bot notifies Lead of auto-approval]

    Promote --> Announce[Bot DMs congratulations<br/>Public announcement posted]
    AutoPromote --> Announce
    Announce --> End([Process complete])

    Decline1 --> End
    Error1 --> End
    AutoReject --> End
    Deny1 --> End
    TrialFail --> End
    VoteFail --> End
    LeadDeny --> End

    classDef successNode stroke:#2e7d32,stroke-width:3px
    classDef failureNode stroke:#c62828,stroke-width:3px

    class Start,Promote,Announce successNode
    class Decline1,Error1,AutoReject,Deny1,TrialFail,VoteFail,LeadDeny,End failureNode
```

### Application Entry Point

The **only** way to enter the community staff pipeline is by emailing shadow@openclaw.ai. Shadow acts as the initial funnel, has a conversation with the candidate, and directs them to the appropriate Team Lead.

### Application Form Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Team | Hidden/Set by bot | ✅ | Which team they're applying for (set by `/onboard-start --team` flag) |
| Timezone | Select/Text | ✅ | Applicant's timezone (for scheduling context) |
| Availability | Text | ✅ | General availability (days/times they're typically free) |
| Motivation | Text (long) | ✅ | Why they want to join this team |
| [Team-specific questions] | Varies | Optional | Teams can add custom form fields |

The form is delivered via Discord DM (modal) after `/onboard-start @user --team [team]` is run.

### Auto-Reject Criteria

- **Minimum server tenure:** User must have been in the OpenClaw Discord for at least 14 days (configurable)
- **Clean record:** No previous bans, kicks, or warnings on record
- **One application at a time:** Cannot have another application pending or in trial (across any team)
- **Cooldown period:** If previously declined or failed trial, must wait 7 days (configurable) before reapplying

### Review Process

- Applications that pass auto-checks are posted as embeds in the team's private channel
- Only **full team members** (not trial members) can approve or deny applications
- **Approval requirements:**
  - Requires **2 full team members** to approve, OR
  - **1 Team Lead** approval (Team Lead can single-handedly approve)
- **Denial:** Any single full team member can deny (immediate denial)
- All decisions are logged with the reviewer's identity and timestamp

### Team-Specific Trial Setup

Each team defines their own trial requirements. The bot provides hooks for teams to:
- Set custom trial duration
- Define success criteria
- Track team-specific metrics
- Provide team-specific onboarding materials

### Trial Requirements

Trial requirements are **team-specific** and defined in team extension specs. The general framework provides:
- Trial start/end tracking
- Configurable trial duration
- Extensible metrics tracking
- Manual or automated completion triggers

Examples of team-specific trial requirements (defined in extensions):
- **VC Mod:** Time slot coverage, voice channel presence tracking (see voice-001-extensions.md)
- **Discord Mod:** Message moderation shadowing, report reviews
- **Helper:** Support ticket shadowing, response quality reviews
- **Configurator:** Configuration task completion, testing procedures

### Vote Mechanics

- Vote only opens AFTER trial requirements are confirmed complete
- Presented as a message with approve/deny buttons in the team channel
- Each full team member gets one vote
- Vote has a defined window (default: 48 hours, configurable per team)
- Threshold: **Simple majority (>50%)** of votes cast
- If no majority by deadline, the application is auto-declined
- If majority approves, application enters **AWAITING_LEAD_APPROVAL** status and is escalated to Team Lead for final sign-off via `/promote` or `/deny` commands
- Team Lead has **7 days** (configurable) to respond after vote passes
- If Team Lead doesn't respond within the timeout window, the promotion is **auto-approved** and Team Lead is notified
- All votes and decisions are logged with voter/actor identity and timestamp

---

## 5. Bot Commands

### Trial Member Commands

| Command | Description |
|---------|-------------|
| `/status` | Shows the trial member's current stage in the pipeline and team-specific progress |
| [Team-specific commands] | Each team may define additional commands for trial members |

### Team Member Commands

| Command | Description |
|---------|-------------|
| `/trials` | List active trials for your team |
| `/trial-status @user` | Shows a trial member's progress (team-specific metrics) |
| [Team-specific commands] | Each team may define additional commands |

### Team Lead Commands

| Command | Description |
|---------|-------------|
| `/onboard-start @user --team [team]` | (Team Lead or Shadow only) Initiates the onboarding process for a user after email funnel |
| `/applications` | Lists pending applications awaiting review for your team |
| `/review <application_id>` | Pulls up a specific application for review |
| `/trial-status @user` | Shows detailed trial progress (all metrics, team-specific data) |
| `/onboarding-stats` | Dashboard: active trials, pending votes, recent outcomes for your team |
| `/promote @user` | (Team Lead only) Final approval for a promotion after team vote passes |
| `/deny @user` | (Team Lead only) Deny a promotion after team vote passes |

### Admin Commands

| Command | Description |
|---------|-------------|
| `/onboarding-config` | Configure system settings (vote window, minimum server tenure, cooldown period, etc.) |
| `/team-config --team [team]` | Configure team-specific settings |

---

## 6. Promotion Pipeline

### Quantitative Gate (Team-Defined)

Each team defines their own quantitative success criteria. The bot provides hooks for teams to register:
- Required metrics
- Minimum thresholds
- Automated checks

Example team criteria:
- **VC Mod:** Minimum voice channel hours, no missed slots
- **Discord Mod:** Minimum messages reviewed, accuracy threshold
- **Helper:** Minimum tickets handled, satisfaction rating
- **Configurator:** Required tasks completed, testing passed

### Qualitative Gate (Team Vote)

- Vote is presented as an embed with ✅ Approve / ❌ Deny buttons
- Each team member votes once (can change their vote before the window closes)
- Vote window: **48 hours** (configurable per team)
- Threshold: **Simple majority (>50%)** of votes cast
- If the vote window expires with no votes → **auto-decline** (applicant can reapply after cooldown)
- If majority approves → escalated to Team Lead for final sign-off via `/promote` or `/deny` commands
- Results are logged and announced in the team channel, mod log channel, and via DM to applicant

---

## 7. Notification System

Notifications are delivered through multiple channels:
- **Bot DMs** to the applicant/trial member for all process updates
- **Team channels** for team-specific notifications
- **Public announcements channel** for promotions
- **Mod log channel** for all events (audit trail)

### Privacy Policy for Trials

**Trial members remain private until promotion is confirmed.** This avoids potential embarrassment if they back out or fail the trial. Trial-related notifications go only to:
- Private team channel
- DMs to the trial member
- Mod log channel (private)

**No public visibility** of trial members until they are promoted to full staff.

### DM Notifications

| Event | DM Content |
|-------|------------|
| Onboarding initiated | "You've been invited to apply for the [Team] team! Please complete the application form." (Sent via modal interaction) |
| Application submitted | "Your application has been received. You'll be notified when it's reviewed." |
| Application auto-rejected | "Your application could not be processed. [Reason: server tenure/previous warnings/cooldown]. You may reapply after [X] days." |
| Application approved | "Congratulations! You've been approved for a [Team] trial. [Guidelines link] [Trial expectations]" |
| Application denied | "Your application has been reviewed and was not approved. You may reapply after [X] days." |
| Trial progress update | "[Team-specific milestone completed] Progress: [team-defined metrics]" |
| Trial failed | "Your trial has ended. [Reason]. You may reapply after [X] days." |
| Trial complete | "You've completed all trial requirements! Your promotion is now under review by the [Team] team." |
| Vote passed, awaiting Lead | "The [Team] team has approved your promotion! Awaiting final sign-off from Team Lead." |
| Auto-approved (Lead timeout) | "🎉 Your promotion has been automatically approved! The Team Lead approval window has expired. Welcome to [Team] Staff!" |
| Promoted (Lead approved) | "🎉 Congratulations! You've been promoted to [Team] Staff. [New permissions overview]" (Also posted in public announcements channel) |
| Declined (vote failed) | "After review, the [Team] team has decided not to proceed with your promotion. You may reapply after [X] days." |
| Declined (Lead denied) | "After final review, your promotion was not approved. You may reapply after [X] days." |

### Team Channel Notifications

**In Team Channel:**
- New application received (after `/onboard-start` and auto-checks) → ping for review
- Trial member progress updates → alerts (team-configurable)
- Trial member failed requirements → alert posted
- Trial member completed all requirements → summary posted + vote opened
- Vote concluded → results announced
- Awaiting Team Lead's final approval → notification with `/promote` and `/deny` instructions and 7-day deadline
- Auto-approval triggered (Lead timeout) → notification posted: "@TeamLead did not respond within 7 days. @User has been auto-approved and promoted to [Team] Staff."

**In Public Announcements Channel:**
- New staff member promoted → celebration announcement with @mention

**In Mod Log Channel (audit trail):**
- All application status changes
- All trial progress updates
- All votes cast
- All role additions and removals

---

## 8. Data Model & Retention

### Core Entities

```
Application {
  id: UUID
  user_id: Discord snowflake
  guild_id: Discord snowflake
  team: string (discord_mod | vc_mod | helper | configurator)
  timezone: string
  availability: string
  motivation: string
  custom_fields: JSON (team-specific application data)
  status: APPLICATION_PENDING_REVIEW | APPLICATION_DENIED | TRIAL_ACTIVE | TRIAL_FAILED | AWAITING_TEAM_VOTE | VOTE_FAILED | AWAITING_LEAD_APPROVAL | PROMOTED_BY_LEAD | PROMOTED_BY_LEAD_INACTION | DENIED_BY_LEAD
  initiated_by: Discord snowflake (tracks who ran /onboard-start)
  approved_by: JSON array of Discord snowflakes (nullable, tracks who approved application - either 2 full members or 1 Team Lead)
  denied_by: Discord snowflake (nullable, tracks who denied application)
  reviewed_at: timestamp (nullable, when application was approved or denied)
  lead_approval_deadline: timestamp (nullable, set when vote passes, used for auto-approve timeout)
  lead_decision_by: Discord snowflake (nullable, Team Lead who made final promotion/denial decision, or who was Lead at time of auto-promotion)
  lead_decided_at: timestamp (nullable, when Lead made decision or when auto-promotion triggered)
  created_at: timestamp
  updated_at: timestamp
}
```

**Application Status Values:**

| Status | Description | Transition From | Transition To |
|--------|-------------|-----------------|---------------|
| `APPLICATION_PENDING_REVIEW` | Application submitted and passed auto-checks, awaiting team review | Initial state after user completes application form | `TRIAL_ACTIVE`, `APPLICATION_DENIED` |
| `APPLICATION_DENIED` | Application denied during initial team review | `APPLICATION_PENDING_REVIEW` | Terminal state |
| `TRIAL_ACTIVE` | User is currently in trial period (bot grants Trial Team role, unlocks channel, sends DM) | `APPLICATION_PENDING_REVIEW` (when team approves) | `TRIAL_FAILED`, `AWAITING_TEAM_VOTE` |
| `TRIAL_FAILED` | User failed to meet trial requirements | `TRIAL_ACTIVE` | Terminal state |
| `AWAITING_TEAM_VOTE` | Trial completed successfully, team promotion vote in progress (48-hour window) | `TRIAL_ACTIVE` (when trial completes) | `VOTE_FAILED`, `AWAITING_LEAD_APPROVAL` |
| `VOTE_FAILED` | Team promotion vote did not reach majority threshold | `AWAITING_TEAM_VOTE` | Terminal state |
| `AWAITING_LEAD_APPROVAL` | Team vote passed, awaiting Team Lead final sign-off (7-day timeout) | `AWAITING_TEAM_VOTE` (when vote passes) | `PROMOTED_BY_LEAD`, `PROMOTED_BY_LEAD_INACTION`, `DENIED_BY_LEAD` |
| `PROMOTED_BY_LEAD` | Team Lead manually approved promotion via `/promote` command | `AWAITING_LEAD_APPROVAL` | Terminal state |
| `PROMOTED_BY_LEAD_INACTION` | Promotion auto-approved after Team Lead did not respond within timeout (7 days) | `AWAITING_LEAD_APPROVAL` | Terminal state |
| `DENIED_BY_LEAD` | Team Lead explicitly denied promotion via `/deny` command after vote passed | `AWAITING_LEAD_APPROVAL` | Terminal state |

**Terminal states** (APPLICATION_DENIED, TRIAL_FAILED, VOTE_FAILED, PROMOTED_BY_LEAD, PROMOTED_BY_LEAD_INACTION, DENIED_BY_LEAD) are final. Users in terminal states can create new applications after the cooldown period expires.

```
Trial {
  id: UUID
  application_id: FK → Application
  user_id: Discord snowflake
  team: string
  start_time: timestamp
  end_time: timestamp (nullable)
  status: ACTIVE | COMPLETED | FAILED
  metrics: JSON (team-specific trial metrics and progress)
  created_at: timestamp
  updated_at: timestamp
}

PromotionVote {
  id: UUID
  application_id: FK → Application
  voter_id: Discord snowflake
  vote: APPROVE | DENY
  created_at: timestamp
  updated_at: timestamp
}

AuditLog {
  id: UUID
  application_id: FK → Application (nullable)
  trial_id: FK → Trial (nullable)
  actor_id: Discord snowflake
  action: string (e.g., "APPLICATION_SUBMITTED", "TRIAL_STARTED", "TRIAL_FAILED", "VOTE_CAST", "PROMOTED", "DECLINED")
  details: JSON
  created_at: timestamp
}
```

### Team Extension Data

Teams can extend the data model with:
- Custom application fields (stored in `Application.custom_fields`)
- Custom trial metrics (stored in `Trial.metrics`)
- Team-specific entities (e.g., TimeSlot for VC Mod)

### Retention Policy

- **All data is retained permanently** (full audit trail)
- No automatic deletion or archival
- Data should be queryable for historical analysis and accountability

---

## 9. Edge Cases & Failure Modes

| Scenario | Behavior |
|----------|----------|
| User emails Shadow but isn't in Discord server | Shadow directs them to join the Discord server first. `/onboard-start` will fail with error if user is not in server. |
| User applies to multiple teams simultaneously | Blocked. Application auto-rejected: "You already have a pending application for [other team]." |
| Trial member leaves server during trial | Auto-fail. Trial marked as TRIAL_FAILED. |
| Trial member tries to apply to different team during active trial | Blocked. "You are currently in a trial for [team]. Complete or withdraw before applying elsewhere." |
| Team members vote after the vote window closes | Vote is rejected. Window is enforced. |
| No team members vote within the window | Auto-decline. Applicant can reapply after cooldown. |
| User who was previously declined tries to reapply within cooldown | `/onboard-start` checks database. If in cooldown, bot notifies initiator: "User must wait [X days] before reapplying." |
| User tries to reapply after cooldown expires | Allowed. Previous application history is logged but does not block new application. |
| Team votes approve but Team Lead never responds | Application enters AWAITING_LEAD_APPROVAL status with 7-day deadline. If Team Lead doesn't respond within 7 days, status changes to PROMOTED_BY_LEAD_INACTION and user is promoted automatically. Team Lead ID is recorded. Team Lead is notified of auto-approval. |
| Team Lead denies after vote passes | Status changes to DENIED_BY_LEAD. Team Lead ID is recorded. Bot removes Trial [Team] role, DMs applicant with decline message. User can reapply after cooldown. |
| Bot goes offline during trial | Trial data persists. **Recommendation:** build in recovery mechanisms to handle missed checks gracefully. |
| User is promoted but already left the server | Promotion fails gracefully. Application marked as completed but role not granted. Logged in audit. |

---

## 10. Guidelines & Documentation

### Documentation Structure

**Primary documentation source:** [openclaw/community](https://github.com/openclaw/community) GitHub repository (public).

**Discord guidelines channels:** Each team has a quick-reference channel with links to the full GitHub docs. The channel contains:
- Links to the full team documentation on GitHub
- Quick command reference (most commonly used commands)
- Biggest rules summary (top violations to watch for)
- Emergency escalation contacts

**Private trial onboarding materials:** Per organizational policy, onboarding trial members is kept private. Trial-specific materials may be hosted in private team channels or DM'd to trial members.

### GitHub Documentation Structure

The openclaw/community repository contains comprehensive documentation for each team:

```
openclaw/community/
├── discord-mod/
│   ├── README.md
│   ├── guidelines.md
│   ├── trial-expectations.md
│   └── scenarios.md
├── vc-mod/
│   ├── README.md
│   ├── guidelines.md
│   ├── trial-expectations.md
│   └── scenarios.md
├── helper/
│   ├── README.md
│   ├── guidelines.md
│   ├── trial-expectations.md
│   └── procedures.md
└── configurator/
    ├── README.md
    ├── guidelines.md
    ├── trial-expectations.md
    └── tasks.md
```

---

## 11. Team-Specific Extensions

This specification provides the **general onboarding framework**. Each team implements additional features and requirements through team-specific extension RFCs:

- **VC Mod Extensions:** `rfc/voice/voice-001-extensions.md`
  - Time slot system
  - Voice channel presence tracking
  - VoiceContributor role system
  - Voice-specific commands

- **Discord Mod Extensions:** `rfc/discord-mod/discord-mod-001-extensions.md` *(planned)*
  - Message review system
  - Moderation action shadowing
  - Case study reviews

- **Helper Extensions:** `rfc/helper/helper-001-extensions.md` *(planned)*
  - Support ticket shadowing
  - Response quality tracking
  - Knowledge base familiarity

- **Configurator Extensions:** `rfc/configurator/configurator-001-extensions.md` *(planned)*
  - Configuration task system
  - Testing procedures
  - Safe deployment practices

---

## Appendix: Configuration Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MIN_SERVER_TENURE_DAYS` | 14 | Minimum days in server before user can apply |
| `REAPPLY_COOLDOWN_DAYS` | 7 | Days user must wait after decline/failure before reapplying |
| `VOTE_WINDOW_HOURS` | 48 | How long the promotion vote stays open (can be overridden per team) |
| `VOTE_THRESHOLD` | 0.5 | Fraction of votes needed to approve (>50%) |
| `DEFAULT_VOTE_OUTCOME` | DECLINE | What happens if no votes are cast in the window |
| `LEAD_APPROVAL_TIMEOUT_DAYS` | 7 | Days to wait for Team Lead approval after vote passes before auto-approving |
| `TRIAL_DURATION_DAYS` | 7 | Default trial duration (can be overridden per team) |
| `COMMUNITY_STAFF_ROLE_ID` | (configured) | Role ID for Community Staff umbrella role (granted on promotion) |
| `PUBLIC_ANNOUNCEMENT_CHANNEL_ID` | (configured) | Channel for promotion announcements |
| `MOD_LOG_CHANNEL_ID` | (configured) | Channel for audit trail logging |
| `DOCS_REPO_URL` | https://github.com/openclaw/community | URL to the primary documentation repository |

### Team-Specific Configuration

Each team has its own configuration namespace:

```
TEAM_CONFIG: {
  discord_mod: {
    channel_id: (configured),
    lead_user_id: (configured),
    trial_duration_days: 7,
    vote_window_hours: 48,
    ... team-specific settings
  },
  vc_mod: {
    channel_id: (configured),
    lead_user_id: (configured),
    trial_duration_days: 7,
    vote_window_hours: 48,
    ... team-specific settings
  },
  helper: { ... },
  configurator: { ... }
}
```

---

## Changelog

### v1.0 (March 1, 2026)
- Initial RFC
- Extracted general framework from voice-specific v1.2 spec
- Designed for extensibility across all four community staff teams
- Defined team extension system
