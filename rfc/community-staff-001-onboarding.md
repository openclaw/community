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
- **KI** — XP and leveling system
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

### Stage 1: Application

```
User emails shadow@openclaw.ai expressing interest in community staff
        │
        ▼
Shadow has initial conversation, determines which team is appropriate
        │
        ├── NOT APPROPRIATE → Shadow declines or suggests different path
        │
        └── APPROPRIATE → Shadow directs to Team Lead
                │
                ▼
        Team Lead runs /onboard-start @user --team [team_name]
                │
                ▼
        Bot checks if user is in Discord server
                │
                ├── NOT IN SERVER → Error: "User must be in the Discord server to onboard"
                └── IN SERVER → Continue
                        │
                        ▼
                Bot DMs user with application form modal
                        │
                        ▼
                User completes form (timezone, availability, motivation, optional: team-specific questions)
                        │
                        ▼
                Bot stores application in database
                        │
                        ▼
                Bot checks auto-reject criteria:
                  - Account must be in server for ≥14 days
                  - No previous bans or warnings on record
                  - No application already pending/in trial (any team)
                  - Not in cooldown period (default: 7 days after decline/failure)
                        │
                        ├── AUTO-REJECTED → Bot DMs applicant (can reapply after cooldown)
                        │                   Bot notifies initiator in team channel
                        │
                        └── ELIGIBLE → Posted to team channel for review
                                │
                                ▼
                        Team reviews and approves/denies
                                │
                                ├── APPROVED → Stage 2
                                └── DENIED → Bot DMs applicant (can reapply after cooldown)
```

**Application Entry Point:**

The **only** way to enter the community staff pipeline is by emailing shadow@openclaw.ai. Shadow acts as the initial funnel, has a conversation with the candidate, and directs them to the appropriate Team Lead.

**Application Form Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| Team | Hidden/Set by bot | ✅ | Which team they're applying for (set by `/onboard-start --team` flag) |
| Timezone | Select/Text | ✅ | Applicant's timezone (for scheduling context) |
| Availability | Text | ✅ | General availability (days/times they're typically free) |
| Motivation | Text (long) | ✅ | Why they want to join this team |
| [Team-specific questions] | Varies | Optional | Teams can add custom form fields |

The form is delivered via Discord DM (modal) after `/onboard-start @user --team [team]` is run.

**Auto-Reject Criteria:**
- **Minimum server tenure:** User must have been in the OpenClaw Discord for at least 14 days (configurable)
- **Clean record:** No previous bans, kicks, or warnings on record
- **One application at a time:** Cannot have another application pending or in trial (across any team)
- **Cooldown period:** If previously declined or failed trial, must wait 7 days (configurable) before reapplying

**Review Process:**
- Applications that pass auto-checks are posted as embeds in the team's private channel
- Team members use approve/deny buttons on the embed
- Any single team member can approve or deny (or: configurable to require multiple approvals)
- Decision is logged with the reviewer's identity and timestamp

---

### Stage 2: Onboarding

```
Application approved
        │
        ▼
Bot grants "Trial [Team]" role
        │
        ▼
Bot unlocks team's private guidelines channel (read-only access)
        │
        ▼
Bot DMs applicant:
  - Confirmation of approval
  - Link to team guidelines channel
  - Instructions on trial expectations (team-specific)
  - Clear warning about trial failure conditions
        │
        ▼
Trial member begins team-specific trial activities → Stage 3
```

**Team-Specific Trial Setup:**

Each team defines their own trial requirements. The bot provides hooks for teams to:
- Set custom trial duration
- Define success criteria
- Track team-specific metrics
- Provide team-specific onboarding materials

---

### Stage 3: Trial Period

```
Trial member completes team-specific trial requirements
        │
        ├── During trial:
        │     - Follows team-specific activities and requirements
        │     - Bot tracks team-defined metrics
        │     - Team members observe and provide feedback
        │
        ├── Failed trial requirements? → TRIAL FAILURE
        │     - Bot removes Trial [Team] role
        │     - Bot DMs: trial failed, can reapply after cooldown
        │     - Application marked as TRIAL_FAILED in database
        │
        └── All requirements completed → Stage 4
```

**Trial Requirements:**

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

---

### Stage 4: Promotion Decision

```
Trial requirements completed (verified by team or bot)
        │
        ▼
Bot confirms trial completion in team channel
Bot opens promotion vote
        │
        ▼
Team members vote (approve/deny)
        │
        ▼
Simple majority (>50%) reached within vote window?
        │
        ├── YES → Vote sent to Team Lead for final sign-off
        │         │
        │         ├── APPROVED → Bot promotes to full [Team] Staff
        │         │               Bot grants Community Staff umbrella role
        │         │               Bot removes Trial [Team] role
        │         │               Bot DMs + public announcement: congratulations
        │         │
        │         └── DENIED → Bot declines
        │                       Bot removes Trial [Team] role
        │                       Bot DMs: declined, can reapply after cooldown
        │
        └── NO → Bot declines
                  Bot removes Trial [Team] role
                  Bot DMs: declined, can reapply after cooldown
```

**Vote Mechanics:**
- Vote only opens AFTER trial requirements are confirmed complete
- Presented as a message with approve/deny buttons in the team channel
- Each full team member gets one vote
- Vote has a defined window (default: 48 hours, configurable per team)
- Threshold: **Simple majority (>50%)** of votes cast
- If no majority by deadline, the application is auto-declined
- If majority approves, escalated to Team Lead for final sign-off via `/promote` or `/deny` commands
- Team Lead can approve or deny at any time (no time limit)
- All votes are logged with voter identity and timestamp

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
| Promoted | "🎉 Congratulations! You've been promoted to [Team] Staff. [New permissions overview]" (Also posted in public announcements channel) |
| Declined (vote failed) | "After review, the [Team] team has decided not to proceed with your promotion. You may reapply after [X] days." |
| Declined (Lead denied) | "After final review, your promotion was not approved. You may reapply after [X] days." |

### Team Channel Notifications

**In Team Channel:**
- New application received (after `/onboard-start` and auto-checks) → ping for review
- Trial member progress updates → alerts (team-configurable)
- Trial member failed requirements → alert posted
- Trial member completed all requirements → summary posted + vote opened
- Vote concluded → results announced
- Awaiting Team Lead's final approval → notification with `/promote` and `/deny` instructions

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
  status: PENDING | APPROVED | DENIED | TRIAL_ACTIVE | TRIAL_FAILED | PROMOTED | VOTE_FAILED
  initiated_by: Discord snowflake (tracks who ran /onboard-start)
  reviewer_id: Discord snowflake (nullable, tracks who approved/denied)
  reviewed_at: timestamp (nullable)
  created_at: timestamp
  updated_at: timestamp
}

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
| Team votes approve but Team Lead never responds | Application stays in "awaiting final approval" state indefinitely. No timeout. |
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
