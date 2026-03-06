# VC Mod Team Extensions — Technical Specification

> **Version:** 1.0
> **Date:** March 1, 2026
> **Status:** RFC - Request for Comments
> **Extends:** [community-staff-001-onboarding.md](../community-staff-001-onboarding.md)
> **Team:** VC Moderators

---

## Table of Contents

1. [Overview](#1-overview)
2. [Time Slot System](#2-time-slot-system)
3. [Voice Presence Tracking](#3-voice-presence-tracking)
4. [VC Mod Commands](#4-vc-mod-commands)
5. [Trial Requirements](#5-trial-requirements)
6. [Promotion Metrics](#6-promotion-metrics)
7. [VoiceContributor Role System](#7-clawtributor-role-system)
8. [Data Model Extensions](#8-data-model-extensions)
9. [Configuration](#9-configuration)

---

## 1. Overview

This specification defines **voice-specific extensions** to the general community staff onboarding framework for the **VC Moderator team**.

### Extension Summary

The VC Mod team requires trial members to:
1. **Claim time slots** for voice channel monitoring (minimum 6 hours)
2. **Be present in voice channels** during claimed slots (75% threshold)
3. **Observe and report** on voice channel activity using `/report`
4. **Cover diverse channels** to demonstrate broad voice moderation awareness

Additionally, VC Mods manage the **VoiceContributor role system**, which grants speaking permissions in protected voice channels to trusted community contributors.

### Integration with General Framework

This spec extends the general framework by:
- Adding custom trial metrics (time slots, presence tracking)
- Defining team-specific commands (`/slots`, `/claim`, `/report`, `/voc-*`)
- Implementing the VoiceContributor nomination/grant system
- Providing quantitative trial completion criteria

---

## 2. Time Slot System

### Overview

Trial VC Mods must claim and complete **time slots** to demonstrate their ability to monitor voice channels consistently.

### Slot Generation

- **Slots are auto-generated 24/7** in 1-hour blocks (00:00–01:00, 01:00–02:00, ..., 23:00–00:00 UTC)
- Slots are generated on a rolling basis (e.g., 2 weeks ahead)
- Each slot is independently claimable
- Multiple trial mods can claim the same slot (slots are not exclusive)

### Claiming Rules

- A trial VC mod must claim a **minimum of 6 hours** (6 slots), with 6-8 hours recommended
- There is no maximum — they can claim more if desired
- Slots cannot overlap for the same trial mod
- A slot can only be unclaimed if it hasn't started yet
- Once a slot's start time passes, it is locked — the trial mod is committed

### Trial Member Commands

| Command | Description |
|---------|-------------|
| `/slots` | Shows available time slots for the current/upcoming week |
| `/claim <slot_id>` | Claims a specific 1-hour time slot |
| `/unclaim <slot_id>` | Unclaims a slot (only allowed before the slot starts) |
| `/my-slots` | Shows all slots the trial mod has claimed |

---

## 3. Voice Presence Tracking

### Tracking Mechanism

- Bot monitors voice state events (`voiceStateUpdate`)
- For each claimed slot, the bot tracks:
  - Whether the trial mod joined any voice channel during the slot window
  - Total time spent in voice channels during the slot
  - **Which specific voice channels** were visited (for diversity assessment)
  - A slot is considered "covered" if the trial mod was present for ≥45 minutes of the 60-minute window (75% threshold, configurable)

### Success Criteria

- **Minimum presence:** 45 minutes out of 60 minutes per slot (75% threshold)
- **Grace period:** 5 minutes late arrival allowed before presence clock starts (configurable)
- **Zero tolerance:** Missing a claimed slot results in **immediate trial failure**

### Slot Monitoring

A scheduled job runs every minute (or on voice state changes) to:
1. Check if any active trial mod has a slot that just ended
2. Verify presence threshold was met
3. If not met → trigger immediate failure flow:
   - Remove Trial VC Mod role
   - DM trial mod: "You missed your claimed slot [time]. Your trial has ended. You may reapply after 7 days."
   - Mark application as TRIAL_FAILED
   - Alert VC Mod team channel
4. If met → mark slot as completed, update progress, DM confirmation

### Failure Conditions

| Scenario | Result |
|----------|--------|
| Trial mod doesn't join any voice channel during slot | MISSED → Immediate trial failure |
| Trial mod joins but leaves early (< 45 min total) | MISSED → Immediate trial failure |
| Trial mod goes offline/loses connection mid-slot | If total presence < 45 min → MISSED → Immediate trial failure |
| Bot goes offline during slot | **Graceful handling:** Don't fail trial mod for bot downtime. Check voice state on reconnect. |

---

## 4. VC Mod Commands

### Trial VC Mod Commands

| Command | Description |
|---------|-------------|
| `/slots` | Shows available time slots for the current/upcoming week |
| `/claim <slot_id>` | Claims a specific 1-hour time slot |
| `/unclaim <slot_id>` | Unclaims a slot (only allowed before the slot starts) |
| `/my-slots` | Shows all slots the trial mod has claimed |
| `/report <user\|message> <reason>` | Logs an incident report during a slot |
| `/status` | Shows trial progress: slots claimed, completed, pending, and reports filed |

### Full VC Mod Commands

| Command | Description |
|---------|-------------|
| `/voc-nominate @user` | Nominate a user for VoiceContributor role (speaking access to protected VCs) |
| `/voc-remove-nomination @user` | Withdraw a nomination you created (only if not yet seconded) |
| `/voc-remove @user` | Vote to remove VoiceContributor role from a user (requires 2 voicemods) |
| `/voc-nominations` | List all pending VoiceContributor nominations |

### VC Mod Team / Lead Commands

| Command | Description |
|---------|-------------|
| `/trial-status @user` | Shows a trial VC mod's progress: slots claimed, completed, reports filed, channels covered |
| `/reports @user` | Shows all reports filed by a specific trial VC mod |

---

## 5. Trial Requirements

### Quantitative Requirements

To proceed to the promotion vote, a trial VC mod must:

- [ ] Claim at least **6 hours** (6 slots) of time slots
- [ ] Complete **all claimed slots** with 75% presence threshold met
- [ ] **Zero missed slots** (zero tolerance)

### Qualitative Expectations

During the trial, the VC mod should:
- **Observe voice channel activity** across diverse channels
- **File reports** using `/report` when they observe noteworthy incidents
  - No minimum report requirement (some slots may be uneventful)
  - Quality and thoughtfulness of reports factor into the team vote
- **Demonstrate awareness** of voice channel dynamics and community behavior

### Trial Completion

Once all claimed slots are completed (bot-verified), the bot:
1. Posts a summary in the VC Mod team channel:
   - Total hours completed
   - Number of reports filed
   - Links to all reports
   - Slot coverage heatmap (what hours/days they covered)
   - **Channel coverage breakdown** (which voice channels were observed)
2. Opens the promotion vote (per general framework)

---

## 6. Promotion Metrics

### Quantitative Metrics (Bot-Verified)

The bot automatically verifies:
- ✅ All claimed slots completed (no missed slots)
- ✅ Minimum 6 hours of coverage
- ✅ Presence threshold met for all slots (≥45 min per slot)

These metrics are displayed in the promotion vote summary.

### Qualitative Metrics (Human-Reviewed)

The VC Mod team reviews:
- **Report quality:** Were reports clear, actionable, and appropriate?
- **Channel diversity:** Did they monitor a variety of voice channels?
- **Judgment:** Do their reports demonstrate good moderation judgment?
- **Activity patterns:** Did they cover peak hours, off-peak hours, or a mix?

---

## 7. VoiceContributor Role System

### Overview

The VoiceContributor role grants speaking permissions in designated protected voice channels. This system allows the VC mod team to recognize and empower trusted community contributors who actively participate in specific voice channels (e.g., Developers, Design, Music Production, etc.).

### Purpose

- **Protected voice channels** are voice channels where only VC mods and VoiceContributors can speak
- Regular members can listen but cannot speak without the VoiceContributor role
- This creates focused, high-signal voice spaces for collaboration

### Who Can Nominate

**IMPORTANT:** Only VC mods (full role) and existing VoiceContributors can nominate users. Regular members cannot nominate.

### Nomination & Grant Process

```
VC mod or VoiceContributor runs /voc-nominate @user
        │
        ▼
Bot checks permissions (must be VC mod or VoiceContributor)
        │
        ├── NOT VC mod/VoiceContributor → Denied: "You don't have permission to nominate."
        │
        └── IS VC mod/VoiceContributor → Continue
                │
                ▼
        Bot posts nomination message in configured channel
        Bot reacts with ✅ emoji
                │
                ▼
        Another VC mod or VoiceContributor reacts with ✅ to second
                │
                ├── Same person tries to second → Denied
                ├── Regular user reacts → Ignored (only VC mod/VoiceContributor reactions count)
                │
                └── Different VC mod/VoiceContributor seconds → Role granted immediately
                        │
                        ▼
                Bot grants VoiceContributor role
                Bot sends DM to recipient
                Bot posts public announcement
                Bot logs to mod log channel
```

### VoiceContributor Commands

| Command | Who Can Use | Description |
|---------|-------------|-------------|
| `/voc-nominate @user` | VC mods & VoiceContributors | Nominate a user for VoiceContributor role |
| `/voc-remove-nomination @user` | Original nominator only | Withdraw a nomination (only if not yet seconded) |
| `/voc-remove @user` | VC mods only | Vote to remove VoiceContributor role (requires 2 VC mods) |
| `/voc-nominations` | VC mods & VoiceContributors | List all pending nominations |

### Role Grant Rules

- **Who can nominate:** ONLY VC mods (full role) or existing VoiceContributors
- **Who can second:** ONLY VC mods or VoiceContributors (except the nominator)
- **Seconding method:** React with ✅ to the nomination message
- **Permission check:** Bot enforces permissions - command will fail if used by non-VC mod/non-VoiceContributor
- **Grant timing:** Immediate upon valid second reaction from VC mod/VoiceContributor
- **Expiration:** Nominations never expire (stay open until seconded or withdrawn)
- **Self-nomination:** Not allowed
- **Duplicate nomination:** Blocked if user already has the role

### Role Removal Process

```
VC mod runs /voc-remove @user
        │
        ▼
Bot posts removal vote in VC mod channel
        │
        ▼
Second VC mod runs /voc-remove @user (or reacts to confirm)
        │
        ▼
Role removed immediately
Bot sends DM to user
Bot logs to mod log channel
```

- **Requires 2 VC mods** to remove (same 2-endorsement pattern)
- Only VC mods can initiate and second removals (VoiceContributors cannot remove)
- Removals are logged permanently in audit trail

### Protected Voice Channels

The VoiceContributor role grants speaking permissions to a configurable set of protected voice channels:
- Channels are defined in bot configuration (`PROTECTED_VOICE_CHANNELS`)
- Can be updated without code changes
- Examples: "Developers", "Design Studio", "Music Production", etc.

**Permission Structure:**
- @everyone: Can join and listen, cannot speak
- VC Mod role: Can speak
- VoiceContributor role: Can speak

### Notifications

| Event | DM | Public Announcement | Mod Log |
|-------|-----|---------------------|---------|
| Nominated | ❌ | ❌ | ✅ |
| Granted VoiceContributor | ✅ | ✅ | ✅ |
| Removed from VoiceContributor | ✅ | ❌ | ✅ |

**DM Templates:**
- **Granted:** "🎉 You've been granted the VoiceContributor role! You can now speak in protected voice channels: [channel list]"
- **Removed:** "Your VoiceContributor role has been removed by the VC mod team."

**Public Announcement Template:**
- "🎉 Welcome @user as a new VoiceContributor! They can now contribute in [protected channels]."

### Edge Cases

| Scenario | Behavior |
|----------|----------|
| User already has VoiceContributor role | Bot blocks nomination: "User is already a VoiceContributor!" |
| User tries to second own nomination | Denied: "You cannot second your own nomination." |
| User tries to nominate themselves | Denied: "You cannot nominate yourself." |
| Nominator withdraws before second | Nomination removed, reaction tracking stopped |
| User leaves server after nomination | Nomination stays open (in case they return) |
| User leaves server after granted role | Role is removed automatically by Discord |
| VoiceContributor nominates another user | Allowed, follows same rules as VC mod nominations |
| Regular member tries `/voc-nominate` | Permission denied: "You don't have permission to nominate." Command not executed. |
| Regular member reacts ✅ to nomination | Reaction is ignored. Only VC mod/VoiceContributor reactions count toward seconding. |

---

## 8. Data Model Extensions

### Additional Entities

```
TimeSlot {
  id: UUID
  application_id: FK → Application (from general framework)
  user_id: Discord snowflake
  start_time: timestamp
  end_time: timestamp
  status: CLAIMED | COMPLETED | MISSED | UNCLAIMED
  presence_minutes: integer (nullable)
  channels_visited: JSON array of channel IDs (tracks which VCs were observed)
  created_at: timestamp
  updated_at: timestamp
}

Report {
  id: UUID
  application_id: FK → Application
  reporter_id: Discord snowflake
  target_type: USER | MESSAGE
  target_id: Discord snowflake
  reason: string
  slot_id: FK → TimeSlot (nullable, links to which slot they were covering)
  created_at: timestamp
}

VoiceContributorNomination {
  id: UUID
  nominee_id: Discord snowflake
  nominator_id: Discord snowflake
  seconder_id: Discord snowflake (nullable, set when seconded)
  status: PENDING | GRANTED | WITHDRAWN
  message_id: Discord snowflake (the nomination message for reactions)
  created_at: timestamp
  seconded_at: timestamp (nullable)
  updated_at: timestamp
}

VoiceContributorRemoval {
  id: UUID
  user_id: Discord snowflake
  initiator_id: Discord snowflake
  seconder_id: Discord snowflake (nullable)
  status: PENDING | COMPLETED | WITHDRAWN
  created_at: timestamp
  completed_at: timestamp (nullable)
  updated_at: timestamp
}
```

### Trial Metrics Extension

The `Trial.metrics` JSON field (from general framework) stores:

```json
{
  "slots_claimed": 6,
  "slots_completed": 6,
  "slots_missed": 0,
  "total_hours": 6.2,
  "reports_filed": 3,
  "channels_covered": [
    "channel_id_1",
    "channel_id_2",
    "channel_id_3"
  ],
  "slot_coverage_heatmap": {
    "Monday": [14, 15, 16],
    "Tuesday": [],
    "Wednesday": [20, 21, 22],
    ...
  }
}
```

---

## 9. Configuration

### VC Mod Team Configuration

```
vc_mod: {
  channel_id: (configured),              # VC Mod team channel
  lead_user_id: (configured),            # VC Mod Lead user ID
  trial_duration_days: 7,                # Trial period length
  vote_window_hours: 48,                 # Promotion vote window

  # Time slot settings
  min_trial_hours: 6,                    # Minimum hours to claim
  slot_duration_minutes: 60,             # Duration of each slot
  presence_threshold_minutes: 45,        # Minutes required to pass a slot (75%)
  grace_period_minutes: 5,               # Late arrival grace period
  slot_generation_days_ahead: 14,        # How far ahead to generate slots

  # VoiceContributor settings
  protected_voice_channels: [            # Array of protected VC IDs
    "channel_id_1",
    "channel_id_2",
    ...
  ],
  nomination_channel_id: (configured),   # Where nominations are posted
}
```

### Additional Configuration Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MIN_TRIAL_HOURS` | 6 | Minimum hours a trial VC mod must claim (6-8 recommended) |
| `SLOT_DURATION_MINUTES` | 60 | Duration of each time slot |
| `PRESENCE_THRESHOLD_MINUTES` | 45 | Minutes present required to "pass" a slot (75% threshold) |
| `GRACE_PERIOD_MINUTES` | 5 | Late arrival grace period before presence clock starts |
| `SLOT_GENERATION_DAYS_AHEAD` | 14 | How far in advance to generate slots |
| `PROTECTED_VOICE_CHANNELS` | (configured) | Array of voice channel IDs where VoiceContributor role grants speak permissions |
| `NOMINATION_CHANNEL_ID` | (configured) | Channel where VoiceContributor nominations are posted |
| `VC_MOD_CHANNEL_ID` | (configured) | Channel ID for VC Mod team channel (review, votes, notifications) |
| `VC_MOD_LEAD_USER_ID` | (configured) | Discord user ID of VC Mod Lead with final promotion authority |

---

## Changelog

### v1.0 (March 1, 2026)
- Initial RFC
- Extracted voice-specific features from general framework
- Defined time slot system
- Defined voice presence tracking
- Defined VoiceContributor role system
- Defined VC mod trial requirements and metrics
