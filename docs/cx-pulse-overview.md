# CX Pulse — What We Built, Why, and How

**Date:** May 3, 2026
**Author:** Gaurav Singh
**Repository:** https://github.com/shipsy/cx-pulse

---

## The Problem

Support operations at Shipsy had no structured daily visibility. Leadership opened DevRev manually, scrolled through tickets, and reacted to fires. There was no way to answer:

- How many tickets are open right now, by which customer?
- Who on the support team is available today, and who's overloaded?
- Which customers are frustrated, and has anyone responded?
- What happened yesterday — did we close more than we opened?
- Is anyone raising issues on WhatsApp or Slack that haven't become tickets yet?

The support team was growing (4 new members joining), but there was no system to distribute work intelligently or track daily performance.

---

## What We Built

### CX Pulse — 3 Automated Routines

| Routine | Schedule | Purpose |
|---------|----------|---------|
| **Morning Brief** | 8 AM IST daily | What's open, who's available, what needs attention today |
| **Evening Wrap-up** | 6 PM IST daily | What got resolved, what didn't, what carries into tomorrow |
| **Sentinel** | Every 2 hours | Real-time alert ONLY if something breaks (war room, incidents, spikes). Silent otherwise. |

All three post to **two Slack channels**:
- Leadership DM (C0A82U7MZ5F)
- #customer-experience-product-support (C07BQD5776Y)

### Auto-Allocation Engine (Phase 2 — Architecture Ready)

When a support ticket arrives in DevRev, automatically route it to the right person based on customer cohort, product pod, team roster, and current workload. Full 466-line architecture plan documented.

---

## Data Sources — Why Each One Matters

### 1. DevRev (Source of Truth)
**What:** All ticket data — open count, SLA status, sentiment, customer cohort, pod assignment, owner.
**Why:** DevRev is where tickets live. The morning brief mirrors **vista-549 ("Support - open tickets")** — the same view leadership uses in the DevRev UI. This ensures the report matches what they'd see if they opened DevRev.
**Key field discovered:** `tnt__customer_cohort_dropdown` — a custom field on every ticket that classifies it into one of 20 cohorts (1-Reliance, 2-Aramex, 3A-On Demand, 4A-B2C Shipper, etc.). This is THE primary routing signal.

### 2. Roster (Google Sheet)
**What:** Who's working today, their shift times, who's on leave.
**Why:** A ticket assigned to someone who's on leave is effectively unassigned. The morning brief cross-references every ticket owner with the roster and flags: "Owner is OFF today — ticket at risk."
**Sheet ID:** `1v8lbH2yZCU7TAInUNO2tqx-HDGbCjPVtqZEWX94pApc`

### 3. Slack Channels (9 monitored + 7 WhatsApp bridges)
**What:** Customer complaints, frustrated alerts, escalations, WhatsApp messages — issues that exist OUTSIDE DevRev.
**Why:** Not every customer issue is a DevRev ticket. Aramex staff message on WhatsApp. Starlinks raises concerns in their external Slack channel. Movin escalates directly. If we only look at DevRev, we miss half the picture.

**Channels scanned daily:**
- `#customer-experience-product-support` — DevRev's automated frustrated customer alerts
- `#support-urgent-help` — manual escalations from CSMs
- `#shipsy-aramex-whatsapp` — Aramex WhatsApp bridge
- `#chronodiali-shipsy-whatsapp` — ChronoDiali WhatsApp bridge
- `#flipkart-shipsy-operation-check-whatsapp` — Flipkart WhatsApp
- `#shipsy-ril-escalation-whatsapp` — Reliance WhatsApp
- `#ext-starlinks-shipsy-support` — Starlinks external channel
- `#ext-qatarpost-qp-shipsy` — Qatar Post external channel
- `#movin` — Movin account channel

### 4. New Relic (Phase 1.5)
**What:** Error rate spikes, latency anomalies, deployment events.
**Why:** When New Relic shows a spike, support tickets follow 15-30 minutes later. If the support team knows about the spike BEFORE tickets arrive, they can prepare.

### 5. GitHub (Phase 1.5)
**What:** Production deploys, mobile build releases.
**Why:** "Was there a deploy yesterday?" is the first question when a new bug surfaces. Including deploy info in the morning brief saves investigation time.
**Scope:** Production deploys and mobile builds only. NOT PRs — hundreds are raised daily, they're noise.

---

## The Morning → Evening Closed Loop

This is the core innovation. Most support reporting is snapshot-based — you see what's open right now. CX Pulse creates **accountability across the day**:

**Morning (8 AM):**
Lists all open tickets as the **Morning Baseline**:
```
TKT-96740 Rozana — GRN Pending | Gangesh | Open
TKT-96957 Flipkart — Auto Checkout | Saurabh | Frustrated
TKT-95550 WF — NTE Staging | SLA BREACHED
... (7 tickets total)
```

**Evening (6 PM):**
Re-checks EVERY morning ticket and reports what happened:
```
Ticket    | Account     | Morning          | Evening
----------|-------------|------------------|-----------------
TKT-96740 | Rozana      | Open             | RESOLVED by Gangesh
TKT-96957 | Flipkart    | Open, Frustrated | RESOLVED by Saurabh
TKT-95550 | WF          | SLA Breached     | STILL OPEN
```
Plus: new tickets that arrived during the day, morning actions status (DONE/NOT DONE), SLA performance.

**Why this matters:** Leadership doesn't have to ask "what happened today?" The answer is in the evening report. And if an action item from the morning wasn't addressed, it shows up as NOT DONE — visible accountability.

---

## Report Format — Sprint-Update Style

Inspired by the Dev Sprint Update format already used at Shipsy. Key elements:

1. **Headlines** — 3 lines with color indicators (🟢🟡🔴). Full picture in 3 seconds.
2. **Code-block tables** — monospace, aligned, scannable. Renders well in Slack.
3. **By Cohort** — ticket count, frustrated count, SLA%, who's on duty.
4. **By Account** — which customers are generating the most load.
5. **Open Tickets** — one-liner per ticket with clickable DevRev link.
6. **Channel Signals** — which channels had activity vs silent.
7. **Team** — workload per person, shift times, ON DUTY vs OFF.
8. **Notable Today** — human insights, not just data.
9. **Actions** — 3-5 specific items: person + ticket + reason.

---

## Cohort Mapping — The Routing Taxonomy

Discovered 20 cohort values from DevRev's `tnt__customer_cohort_dropdown` field:

| Tier | Cohorts | Team Pool |
|------|---------|-----------|
| Tier 1 (Dedicated) | 1-Reliance, 1-DTDC | Deepanshu, Vikas, Asif, Medha, Vidushi, Madhav |
| Tier 2 (Strategic) | 2-Aramex, 2-HNK | Kaustuv, Laxmi, Srijan |
| 3PL | 4A-B2C Shipper, 4A-B2C LSP, 5A-B2B LSP, 5A-B2B Shipper | Gaurav, Gangesh, Omi (new), Pakhi (new) |
| On-Demand | 3A-On Demand, 3B-S (On Demand) | Saurabh, SKG (new), Tejal (new) |
| Other | WMS, Exim, Platform, Roadmap, AI, TBD | Varies |

**Why this structure:** Analyzed 12+ real tickets. Found that cohort determines the team (Reliance tickets always go to Reliance people, regardless of feature). Pod refines within the team (within On-Demand, "On Demand" vs "MCM" pod means different people).

---

## Key Findings from Data Analysis

### 1. 3PL is the most overwhelmed section
- 50% first-response SLA miss rate
- 7 tickets across 5 orgs handled by 3 people
- Gangesh alone had 3 tickets across 3 different orgs
- **Action taken:** Omi + Pakhi assigned to 3PL, Gaurav joins to add senior capacity

### 2. On-Demand was a single point of failure
- Saurabh handled ALL Flipkart tickets alone
- First response SLA missed at 419 minutes
- **Action taken:** SKG + Tejal assigned to On-Demand

### 3. Aramex has 23+ day average resolution
- Consistent SLA misses on resolution time
- Configuration and mapping issues taking weeks

### 4. Reliance generates 25% of all volume
- But is well-staffed with dedicated team
- Stable, not a risk area

### 5. 88% of tickets are severity "medium"
- Severity field is underutilized — everything defaults to medium
- Severity is NOT useful for routing (but useful for priority within a queue)

### 6. 15 frustrated customer alerts fired in a single day
- Rozana appeared in 5 of 15 — recurring pattern
- 2 alerts were for UNASSIGNED tickets — nobody was looking at them

---

## Auto-Allocation Architecture (Phase 2)

Full 466-line plan documented at `docs/auto-allocation-architecture.md`. Summary:

```
Ticket Arrives (DevRev webhook)
  → Step 0: Skip spam and non-Support tickets
  → Step 1: Determine Cohort (tnt__customer_cohort_dropdown or email domain fallback)
  → Step 2: Determine Pod (tnt__pod or keyword scan of title + body)
  → Step 3: Select Person (cohort+pod → pool → filter by roster → least-loaded)
  → Step 4: Assign (set owner + group + comment explaining why + Slack DM)
```

**Design principles:**
- Cohort as Level 1 because it's the Goldilocks abstraction (not too granular like accounts, not too coarse like products)
- Pod as Level 2 because it maps to engineering team structure
- Load-balanced round-robin (not pure round-robin) because some tickets take days
- Roster integration because assigning to someone on leave = wasting time
- 8 edge cases handled: spam, unknown accounts, reopened tickets, blocker off-hours, cross-pod, duplicates

---

## Technical Implementation

### Repository Structure
```
cx-pulse/
├── CLAUDE.md                              — Project overview and conventions
├── .mcp.json                              — DevRev, Slack, Google Drive, GitHub, New Relic
├── .claude/
│   ├── skills/
│   │   ├── morning-brief/SKILL.md         — 8 AM daily (mirrors vista-549)
│   │   ├── evening-wrap/SKILL.md          — 6 PM daily (morning→evening closed loop)
│   │   ├── sentinel/SKILL.md              — Every 2h proactive alerts
│   │   └── allocate-ticket/SKILL.md       — Auto-allocation logic
│   ├── agents/
│   │   └── deep-research.md               — Isolated ticket pattern analysis
│   └── rules/
│       ├── cohort-mapping.md              — 20 cohort values + email domain fallbacks
│       ├── team-pools.md                  — (cohort, pod) → people with DevRev IDs
│       └── devrev-api.md                  — Stage IDs, custom fields, API calls
├── config/
│   └── thresholds.json                    — Capacity limits, SLA targets, business hours
└── docs/
    ├── cx-pulse-overview.md               — This document
    └── auto-allocation-architecture.md    — Full allocation plan (466 lines)
```

### Claude Code Remote Triggers
| Trigger | Cron (UTC) | Local Time | ID |
|---------|-----------|------------|-----|
| Morning Brief | `30 2 * * *` | 8:00 AM IST | trig_014fPFAaz4hD9VnrATK5ht34 |
| Evening Wrap-up | `30 12 * * *` | 6:00 PM IST | trig_01TxAg9cQntgoJYcQPWtrR5j |
| Sentinel | `7 */2 * * *` | Every 2 hours | trig_01ADRkfxMnQetpgU2FWnj7N9 |

### MCP Connections
- DevRev — ticket data, SLA, sentiment, cohorts
- Slack — read 16+ channels, post reports, send DMs
- Google Drive — roster sheet
- GitHub — deploy tracking (Phase 1.5)
- New Relic — anomaly detection (Phase 1.5)

---

## Roadmap

| Phase | What | Status |
|-------|------|--------|
| Phase 1 | Morning + Evening + Sentinel routines | **LIVE** |
| Phase 1.5 | New Relic spikes + GitHub deploys in reports | Ready to add |
| Phase 2 | Auto-allocation engine | Architecture complete, build next week |
| Phase 3 | Differentiated SLA per cohort (Tier 1 tighter than Tier 4) | Planning |
| Phase 4 | ML-based pod prediction from ticket title/body | Future |

---

## How This Connects to Shipra

CX Pulse is a sibling project to Shipra (the AI support investigation agent). They complement each other:

- **Shipra** investigates individual tickets — generates RCA, suggests responses
- **CX Pulse** operates at the fleet level — daily visibility, workload management, proactive alerting

When auto-allocation is live (Phase 2), the flow will be:
```
Ticket arrives → CX Pulse auto-assigns to the right person
              → Shipra auto-investigates and generates RCA
              → Assignee gets: ticket + RCA + context in one Slack DM
```

This is the support operations flywheel: intelligent routing + intelligent investigation + daily accountability.
