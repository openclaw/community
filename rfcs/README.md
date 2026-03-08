# Community Improvement Plans (CIPs)

This directory contains Community Improvement Plans — formal proposals for changes that affect the OpenClaw Discord community.

## Process

1. **Pitch:** Open an [RFCIP issue](https://github.com/openclaw/openclaw-discord-community/issues/new?template=rfcip.yml) describing the problem and your proposed direction.
2. **Champion:** Staff reviews the RFCIP and assigns a champion (staff sponsor and advocate). The champion assigns the next available CIP number.
3. **Draft:** Once greenlit, open a pull request adding `rfcs/XXXX-slug.md` (where `XXXX` is the PR number). Use [`0003-rfcip-template.md`](0003-rfcip-template.md) as your starting template.
4. **Discussion:** Minimum 7-day discussion period on the GitHub PR and (optionally) Discord. The champion chooses the Discord venue.
5. **Decision:** The Admin makes the final accept or reject decision.
6. **Accepted:** PR is merged. A tracking issue is opened for implementation.
7. **Rejected:** PR is closed with rationale documented.

## Naming

CIP numbers are sequential, assigned by the champion when the RFCIP is greenlit. File names follow the pattern:

```
rfcs/XXXX-short-descriptive-slug.md
```

## Lifecycle Labels

| Label | Meaning |
| --- | --- |
| `cip:draft` | Initial RFCIP or early CIP PR |
| `cip:discussion` | Under active community review (7-day minimum) |
| `cip:accepted` | Approved by Admin, PR merged |
| `cip:rejected` | Not approved |

## Type Labels

| Label | Meaning |
| --- | --- |
| `cip:governance` | Rules, policies, processes |
| `cip:structural` | Channels, roles, team changes |
| `cip:tooling` | Bots, automation, integrations |
| `cip:events` | Programs, activities, community events |

## Key Rules

- **Anyone** in the community can author a CIP, but a **staff champion** is required.
- Authors may **withdraw** their CIP at any time before acceptance.
- Changes to **accepted CIPs** require a new CIP that supersedes the original.
- **AI-assisted drafting** is permitted but must be disclosed.

## Template

- [`0003-rfcip-template.md`](0003-rfcip-template.md) — CIP-003: the formal specification of the CIP process
- [`XXXX-cip-blank-template.md`](XXXX-cip-blank-template.md) — Blank template to copy when starting a new CIP
