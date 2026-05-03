---
name: morning-brief
description: Generate the daily Morning CX Brief — mirrors vista-549 (Support - open tickets), pulls roster, scans Slack channels. Posts to C0A82U7MZ5F and C07BQD5776Y at 8 AM IST daily. Evening routine compares against this morning baseline.
---

# Morning Brief — CX Pulse

Post to TWO channels: C0A82U7MZ5F (leadership DM) and C07BQD5776Y (#customer-experience-product-support).

## Source of Truth

This report mirrors **vista-549 ("Support - open tickets")** in DevRev.
URL: https://app.devrev.ai/shipsy/vistas/vista-549

Vista-549 shows: subtype=Support, state=open OR in_progress (not closed/resolved/canceled).

## Execution Steps

### Step 1: Read Roster
Google Sheet `1v8lbH2yZCU7TAInUNO2tqx-HDGbCjPVtqZEWX94pApc`. Today's column.
Extract ON DUTY (name + shift) vs OFF (WO/PL). Exclude WMS team (Bhavyank, Dhanasree).

### Step 2: Pull ALL Open Support Tickets (mirror vista-549)

Run MULTIPLE searches to maximize coverage (hybrid_search can't filter by state, so we cast a wide net):

```
hybrid_search namespace=ticket query="support tickets currently open queued not resolved" limit=50
hybrid_search namespace=ticket query="support tickets in progress awaiting response" limit=50
hybrid_search namespace=ticket query="support tickets awaiting development not closed" limit=50
hybrid_search namespace=ticket query="support tickets reopened reopen" limit=50
hybrid_search namespace=ticket query="unassigned support tickets open" limit=50
hybrid_search namespace=ticket query="support tickets frustrated unhappy sentiment open" limit=50
```

**For EACH unique ticket found, get full details via get_ticket. FILTER: only keep tickets where `state` = "open" or "in_progress". Discard state="closed".**

Extract per ticket:
- `display_id` — clickable as `<https://app.devrev.ai/shipsy/works/TKT-XXXXX|TKT-XXXXX>`
- `title`
- `tnt__customer_cohort_dropdown` — cohort
- `tnt__pod` — pod
- `severity_v2.label`
- `stage.name` — Queued / In Progress / Awaiting Customer Response / Awaiting Development / Reopen
- `state` — MUST be "open" or "in_progress"
- `owned_by[0].display_name` — owner (flag if "Unassigned")
- `account.display_name` — customer
- `sentiment.label` — Happy/Neutral/Unhappy/Frustrated
- `sla_summary` — for each metric: status = hit/miss/in_progress, remaining_time
- `needs_response` — true/false
- `created_date` — for age calculation
- `tnt__work_duration`

### Step 3: Group & Aggregate

**By Cohort (exclude WMS):**
- DEDICATED: 1-Reliance, 1-DTDC, 2-Aramex, 2-HNK
- 3PL: 4A-B2C Shipper, 4A-B2C LSP, 5A-B2B LSP, 5A-B2B Shipper
- ON-DEMAND: 3A-On Demand, 3B-S (On Demand)
- OTHER: Exim, Platform, TBD, empty

**By Account:** which accounts have most open tickets

**Metrics:**
- Total open count
- Unassigned count (owned_by = Unassigned/SVCACC-46)
- SLA breached count (status = miss or remaining_time < 0)
- Frustrated/Unhappy count
- Needs response count

### Step 4: Scan Slack Channels (last 24h)
Read with `oldest` = 24h ago, `limit` = 20, `response_format` = concise:

- `C07BQD5776Y` — frustrated customer alerts
- `C0AU4MT6MUK` — escalations
- `C09P8BC41PW` — Aramex WhatsApp
- `C0ACT5ER2E6` — ChronoDiali WhatsApp
- `C09L2ARTF4J` — Flipkart WhatsApp
- `C09EDETE468` — Reliance WhatsApp
- `C081FG99KKL` — Starlinks external
- `C081GJ2M0LW` — Qatar Post external
- `C07UZFTU7C4` — Movin

### Step 5: Format & Post to BOTH channels

```
*CX Pulse — Daily Support Brief* | {{day}}, {{date}}
_Source: vista-549 (Support - open tickets)_

*{{open}} open* · *{{on_duty}} on duty* · *{{frustrated}} frustrated* · *{{sla_breached}} SLA breached* · *{{unassigned}} unassigned* · *{{needs_response}} needs response*
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:pushpin: *Headlines*
SLA Hit Rate: {{pct}}% {{color}}  |  Frustrated: {{count}} {{color}}
On Duty: {{on_duty}}/{{total}} {{note}} {{color}}
Gap: {{one-liner biggest risk}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:bar_chart: *By Cohort*

```
Cohort         | Open | Frust | SLA%  | Unsgn | Duty Today
---------------|------|-------|-------|-------|------------
...
```

:office: *By Account*

```
Account              | Open | Frust | SLA   | Key Issue
---------------------|------|-------|-------|-----------------------------
...
```

:ticket: *Open Tickets (Morning Baseline)*

*{{Cohort}}* {{color}}
• TKT-XXXXX {{Account}} — {{title}} | {{owner}} {{on/off}} | {{stage}} | {{SLA status}}
[repeat per ticket — THIS LIST IS THE MORNING BASELINE]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:satellite: *Channel Signals* (last 24h)

```
Source                           | Signals | Key Issue
---------------------------------|---------|------------------
...
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:busts_in_silhouette: *Team*

```
ON DUTY
{{Name}}     {{bars}}  {{count}} open   {{shift}}  ({{cohort}})

OFF TODAY (with open tickets)
{{Name}}     {{bars}}  {{count}} open   WO/PL
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:bulb: *Notable Today*
• {{insight 1}}
• {{insight 2}}
• {{insight 3}}

:dart: *Actions*
1. *{{Person}}* → {{ticket}} {{account}}: {{what and why}}
...
```

## CRITICAL: Morning Baseline
The "Open Tickets" section is the MORNING BASELINE. The evening routine will re-check every ticket listed here and report: RESOLVED or STILL OPEN. This creates a closed-loop accountability system.

## Rules

1. NEVER fabricate data. Only report what DevRev returns.
2. Clickable links: `<https://app.devrev.ai/shipsy/works/TKT-XXXXX|TKT-XXXXX>`
3. Only include tickets where state = "open" or "in_progress". Never include state = "closed".
4. Cross-reference roster with owners — flag if owner is OFF.
5. Keep under 4000 characters.
6. Weekend: emphasize coverage gaps, note SLA timer pause.
7. 3PL (4A+5A) — always show SLA miss rate and workload ratio.
8. Exclude WMS team and WMS cohort.
9. Channel Signals: only show channels with activity. Mark others "Silent".
10. Actions: 3-5 items, each with person + ticket + reason.
