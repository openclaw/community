# CIP-003: Community Improvement Plan Process

| Field                   | Value                                                        |
| ----------------------- | ------------------------------------------------------------ |
| **CIP**                 | 003                                                          |
| **Title**               | Community Improvement Plan Process                           |
| **Author(s)**           | balaa                                                        |
| **Champion**            | —                                                            |
| **Status**              | Draft                                                        |
| **Type**                | Governance                                                   |
| **Date**                | 2026-03-07                                                   |
| **RFCIP Issue**         | —                                                            |
| **Teams Affected**      | All teams                                                    |
| **Discussion (GitHub)** | —                                                            |
| **Discussion (Discord)**| —                                                            |
| **Supersedes**          | —                                                            |
| **Superseded by**       | —                                                            |

---

## Summary

This CIP establishes the Community Improvement Plan process — a formal, community-led proposal system for changes that affect the OpenClaw Discord community. It defines the two artifacts (RFCIP and CIP), their structure, lifecycle, roles, and rules. All future CIPs are written against the template and process defined here.

## Motivation

The OpenClaw Discord community currently has lightweight GitHub issue templates for rules changes and policy changes. These work well for small, staff-driven tweaks, but there is no structured path for community members to propose and shepherd larger initiatives — things like channel restructuring, new tooling, event programs, or cross-team process changes.

Without a formal proposal process:

- Community-driven ideas lack a clear path from concept to implementation.
- Proposals vary wildly in quality and completeness, making them hard to evaluate.
- There is no public record of what was proposed, why it was accepted or rejected, and what alternatives were considered.
- Staff has no structured way to sponsor and advocate for community ideas.

A lightweight RFC-inspired process fills this gap while keeping overhead appropriate for a Discord community, not a protocol specification body.

## Detailed Design

### Two Artifacts

The CIP process involves two artifacts:

#### 1. RFCIP (Request for Community Improvement Plan)

A GitHub issue opened using the RFCIP issue template. This is a **mandatory screening gate** — every CIP must start as an RFCIP. Its purpose is to pitch an idea at low cost before investing in a full proposal.

**RFCIP fields:**

| Field                | Required | Description                                              |
| -------------------- | -------- | -------------------------------------------------------- |
| Problem statement    | Yes      | What problem, gap, or opportunity this addresses         |
| Proposed direction   | Yes      | High-level sketch of the proposed solution               |
| Scope and impact     | Yes      | What parts of the community are affected                 |
| CIP type             | Yes      | Governance, Structural, Tooling, Events, or Other        |
| Willingness to author| Yes      | Whether the submitter will write the full CIP            |
| Proposed champion    | No       | Preferred staff sponsor, if any                          |
| Prior discussion     | No       | Links to relevant Discord threads or issues              |
| AI disclosure        | No       | How AI tools were used, if applicable                    |

**Checklist (all required):**
- Searched existing issues and CIPs to avoid duplicates.
- Believes this requires a full CIP (not a simple rules or policy change).
- If AI tools were used, has disclosed that.

#### 2. CIP (Community Improvement Plan)

A formal markdown document submitted as a pull request to the `rfcs/` directory. The CIP number is **sequentially assigned by the champion** when the RFCIP is greenlit. The file is named `rfcs/XXXX-short-descriptive-slug.md`.

**CIP metadata header:**

| Field                   | Description                                               |
| ----------------------- | --------------------------------------------------------- |
| CIP                     | Sequentially assigned number                              |
| Title                   | Descriptive title                                         |
| Author(s)               | Name(s) and Discord handle(s)                             |
| Champion                | Staff sponsor — name and Discord handle                   |
| Status                  | Draft, Discussion, Accepted, or Rejected                  |
| Type                    | Governance, Structural, Tooling, or Events                |
| Date                    | Date of initial submission (YYYY-MM-DD)                   |
| RFCIP Issue             | Link to the originating RFCIP issue                       |
| Teams Affected          | Which teams are impacted                                  |
| Discussion (GitHub)     | Link to the CIP pull request                              |
| Discussion (Discord)    | Link to Discord thread, if any                            |
| Supersedes              | CIP number this replaces, if applicable                   |
| Superseded by           | CIP number that replaced this, if applicable              |

**CIP document sections:**

| #  | Section              | Purpose                                                                |
| -- | -------------------- | ---------------------------------------------------------------------- |
| 1  | Summary              | One-paragraph overview of the proposal                                 |
| 2  | Motivation           | The problem or opportunity; concrete examples and evidence              |
| 3  | Detailed Design      | The core proposal in full detail; before/after states where applicable  |
| 4  | Drawbacks            | Honest costs, risks, and trade-offs                                    |
| 5  | Alternatives         | Other approaches considered and why they were rejected                  |
| 6  | Prior Art            | Examples from other communities or platforms                            |
| 7  | Implementation Plan  | Concrete steps, responsibilities, and required changes                  |
| 8  | Timeline             | Proposed schedule and milestones                                        |
| 9  | Success Metrics      | Measurable or observable outcomes that define success                   |
| 10 | Unresolved Questions | Open items needing further discussion or investigation                  |

An **AI Disclosure** section is included at the end when AI tools were used in drafting.

### Lifecycle

A CIP moves through three states:

```
Draft ──▶ Discussion ──▶ Accepted
                    └──▶ Rejected
```

- **Draft:** The CIP pull request has been opened. The author and champion are refining the document.
- **Discussion:** The CIP is ready for community review. A **minimum 7-day discussion period** begins. Discussion happens on both the GitHub pull request (formal review) and Discord (informal discussion). The champion chooses the Discord venue per-CIP.
- **Accepted:** The Admin has approved the CIP. The pull request is merged and a tracking issue is opened for implementation.
- **Rejected:** The Admin has declined the CIP. The pull request is closed with rationale documented.

### Process Flow

```
Community member
      │
      ▼
┌─────────────┐
│ Open RFCIP  │  ◀── GitHub issue using RFCIP template
│   issue     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Staff review│  ◀── Staff evaluates the pitch
│ + assign    │
│  champion   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Champion    │  ◀── Champion greenlights + assigns CIP number
│ greenlights │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Open CIP PR │  ◀── PR adding rfcs/XXXX-slug.md
│  (Draft)    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Discussion  │  ◀── 7-day minimum; GitHub + Discord
│  period     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Admin     │
│  decision   │
└──┬──────┬───┘
   │      │
   ▼      ▼
Accepted  Rejected
 (merge)  (close + rationale)
   │
   ▼
Tracking issue
  opened
```

### Roles

#### Author
- Any community member.
- Writes the RFCIP and (if willing) the full CIP document.
- Responds to feedback and revises the proposal during discussion.

#### Champion
- A staff member who serves as **sponsor and advocate** for the proposal.
- **Assigns the CIP number** when greenlighting the RFCIP (next available sequential number).
- Actively advocates for the proposal to the Admin and team leads.
- Takes ownership of shepherding the CIP through the process.
- Chooses the Discord discussion venue.
- Posts periodic summaries if discussion spans both platforms.

#### Admin
- Makes the **final accept or reject decision** on every CIP.
- Consistent with the existing community hierarchy.

### Numbering

CIP numbers are **sequential and manually assigned by the champion** when the RFCIP is greenlit. The champion determines the next available number before the author opens the CIP pull request. The file is named:

```
rfcs/XXXX-short-descriptive-slug.md
```

For example, if the next available number is 004:
```
rfcs/0004-voice-channel-restructure.md
```

The champion is responsible for checking existing CIPs to avoid number collisions.

### GitHub Labels

**Status labels** track lifecycle state:

| Label            | Meaning                                |
| ---------------- | -------------------------------------- |
| `cip:draft`      | Initial RFCIP or early CIP PR         |
| `cip:discussion` | Under active community review          |
| `cip:accepted`   | Approved by Admin, PR merged           |
| `cip:rejected`   | Not approved                           |

**Type labels** categorize proposals:

| Label             | Meaning                               |
| ----------------- | ------------------------------------- |
| `cip:governance`  | Rules, policies, processes            |
| `cip:structural`  | Channels, roles, team changes         |
| `cip:tooling`     | Bots, automation, integrations        |
| `cip:events`      | Programs, activities, community events|

### Withdrawal

An author may withdraw their CIP at any time before acceptance by closing their pull request. No approval is needed.

### Amending Accepted CIPs

Changes to accepted CIPs require a **new CIP that supersedes** the original. The new CIP sets `Supersedes: CIP-XXXX` in its header, and the original CIP is updated to set `Superseded by: CIP-YYYY`. In-place edits to accepted CIPs are not permitted.

### AI Policy

AI-assisted drafting is permitted for both RFCIPs and CIPs, but **must be disclosed**. The RFCIP issue template includes an AI disclosure field. The CIP document template includes an AI Disclosure section. The focus is on content quality and transparency, not policing the tools used.

### Relationship to Existing Templates

The CIP process runs **in parallel** with the existing rules-change and policy-change issue templates. Those templates remain the appropriate path for small, targeted changes to rules or policies. CIPs are for larger, structural, or cross-cutting proposals that benefit from the full RFC treatment.

## Drawbacks

- **Process overhead.** For a Discord community, even a lightweight RFC process adds friction. Some good ideas may not get proposed because the process feels heavy.
- **Champion bottleneck.** Requiring a staff champion means proposals can stall if no staff member is available or interested in sponsoring.
- **Low volume risk.** If CIPs are rarely used, the process becomes dead infrastructure that confuses newcomers.
- **Parallel processes.** Having both lightweight issue templates and the CIP process may create ambiguity about which path to use.

## Alternatives

- **Expand existing issue templates.** Add more fields to the rules/policy change templates instead of creating a new process. Rejected because the scope of CIPs is much broader than rules/policy, and the proposal-review-accept lifecycle doesn't map to a single issue template.
- **Discord-only proposals.** Use Discord threads or a forum channel for proposals without GitHub. Rejected because GitHub provides better long-term record-keeping, version control, and structured review.
- **Wiki-based proposals.** Use the GitHub wiki. Rejected because wikis lack the PR-based review workflow that makes Rust RFCs effective.
- **No formal process.** Continue with ad-hoc community suggestions. Rejected because this provides no structured path from idea to implementation and no public record of decisions.

## Prior Art

- **Rust RFCs** ([rust-lang/rfcs](https://github.com/rust-lang/rfcs)): PR-based, discussion-heavy, merged when accepted. Primary inspiration for the CIP process. Adapted from a programming language context to a community governance context.
- **Python PEPs** ([python/peps](https://github.com/python/peps)): Numbered proposals with an editor role and formal lifecycle. More heavyweight than needed here.
- **Ethereum EIPs** ([ethereum/EIPs](https://github.com/ethereum/EIPs)): Typed proposals (Standards, Meta, Informational) with clear status flow. The type categorization (Governance, Structural, Tooling, Events) is loosely inspired by EIP types.
- **IETF RFCs**: Highly formal, consensus-driven. Too heavyweight for a Discord community, but the concept of a numbered, permanent record of decisions is valuable.

## Implementation Plan

1. **Merge this CIP** into the `rfcs/` directory as the foundational process document.
2. **Create GitHub labels**: `cip:draft`, `cip:discussion`, `cip:accepted`, `cip:rejected`, `cip:governance`, `cip:structural`, `cip:tooling`, `cip:events`.
3. **Deploy the RFCIP issue template** (`.github/ISSUE_TEMPLATE/rfcip.yml`).
4. **Create a blank CIP template file** for authors to copy when starting a new CIP.
5. **Update the repository README** to reference the CIP process and link to `rfcs/README.md`.
6. **Announce the CIP process** in the Discord #announcements channel.
7. **Brief staff** on the champion role and expectations.

## Timeline

- **Week 1:** Merge CIP-003 and supporting files. Create labels. Update README.
- **Week 2:** Announce to the community. Brief staff on the process.
- **Ongoing:** Iterate on the process based on experience with the first few CIPs.

## Success Metrics

- At least **3 RFCIPs** submitted within the first 3 months.
- At least **1 CIP accepted** within the first 3 months.
- Community members report the process is **clear and accessible** (qualitative feedback).
- Staff champions report the workload is **manageable** (qualitative feedback).
- No CIPs stall indefinitely waiting for a champion (champion assigned within 7 days of RFCIP submission).

## Unresolved Questions

- Should there be a maximum time limit for the discussion period, or can it stay open indefinitely?
- Should deferred CIPs (good idea, wrong time) have a formal status, or is rejection with a note sufficient?
- How should CIPs that span multiple types be labeled?
- Should the community be able to see a dashboard or index of all CIPs and their statuses?
- What happens if a champion leaves staff mid-process — is a new champion assigned automatically?

---

## AI Disclosure

AI tools (Claude) were used as a collaborative partner in designing the CIP process structure, drafting this document, and generating the RFCIP issue template. All design decisions were made by the author through a structured questionnaire process.

---

*This is the foundational CIP that defines the Community Improvement Plan process.
To propose a CIP, start by opening an [RFCIP issue](https://github.com/openclaw/openclaw-discord-community/issues/new?template=rfcip.yml).*
