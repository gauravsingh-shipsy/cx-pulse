---
name: morning-brief
description: Generate the daily Morning CX Brief — pulls DevRev tickets, roster from Google Sheet, Slack channel signals. Posts to C0A82U7MZ5F and C07BQD5776Y at 8 AM IST daily.
---

# Morning Brief — CX Pulse

Post to TWO channels: C0A82U7MZ5F (leadership DM) and C07BQD5776Y (#customer-experience-product-support).

## Execution Steps

### Step 1: Read Roster
Google Sheet `1v8lbH2yZCU7TAInUNO2tqx-HDGbCjPVtqZEWX94pApc`. Today's column.
Extract ON DUTY (name + shift) vs OFF (WO/PL). Exclude WMS team (Bhavyank, Dhanasree).

### Step 2: Pull DevRev Tickets
- `hybrid_search namespace=ticket query="all open support tickets not resolved not closed" limit=50`
- `hybrid_search namespace=ticket query="support tickets created in last 24 hours" limit=50`
- `hybrid_search namespace=ticket query="unassigned support tickets" limit=50`

Get details via `get_ticket`. Extract: display_id, title, tnt__customer_cohort_dropdown, severity, stage, owned_by, account, sentiment, sla_summary.

Group by cohort: DEDICATED (1-Reliance, 1-DTDC, 2-Aramex, 2-HNK), 3PL (4A+5A), ON-DEMAND (3A), OTHER.
Exclude WMS cohort.

### Step 3: Scan Slack Channels (last 24h)
- `C07BQD5776Y` — frustrated customer alerts
- `C0AU4MT6MUK` — escalations
- `C09P8BC41PW` — Aramex WhatsApp
- `C0ACT5ER2E6` — ChronoDiali WhatsApp
- `C09L2ARTF4J` — Flipkart WhatsApp
- `C09EDETE468` — Reliance WhatsApp
- `C081FG99KKL` — Starlinks external
- `C081GJ2M0LW` — Qatar Post external
- `C07UZFTU7C4` — Movin

### Step 4: Format & Post to BOTH channels

```
*CX Pulse — Daily Support Brief* | {{day}}, {{date}}

*{{open}} open* · *{{on_duty}} on duty* · *{{frustrated}} frustrated alerts* · *{{sla_breached}} SLA breached* · *{{unassigned}} unassigned*
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:pushpin: *Headlines*
SLA Hit Rate: {{pct}}% {{color}}  |  Frustrated: {{count}} alerts {{color}}
On Duty: {{on_duty}}/{{total}} {{weekend note if applicable}} {{color}}
Gap: {{one-liner about biggest risk today}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:bar_chart: *By Cohort* — {{color per cohort}}

```
Cohort         | Open | Frust | SLA%  | Duty Today
---------------|------|-------|-------|------------
3PL (4A+5A)    | ...  | ...   | ...   | {{names or "None"}}
On-Demand (3A) | ...  | ...   | ...   | ...
Dedicated      | ...  | ...   | ...   | ...
```

:office: *By Account* — top ticket generators

```
Account              | Open | Frust | Key Issue
---------------------|------|-------|-----------------------------
{{top accounts}}
```

:ticket: *Open Tickets*

*{{Cohort}}* {{color}}
> <https://app.devrev.ai/shipsy/works/TKT-XXXXX|TKT-XXXXX> {{Account}} — {{title}} | {{owner}} {{on duty?}} | {{SLA status}}
[repeat per ticket]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:satellite: *Channel Signals* (last 24h)

```
Source                          | Signals | Key Issue
--------------------------------|---------|------------------
{{channel}}                     | {{cnt}} | {{summary or "Silent"}}
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:busts_in_silhouette: *Team*

```
ON DUTY
{{Name}}     {{bars}}  {{count}} open   {{shift}}  ({{cohort}})

OFF TODAY (with open tickets)
{{Name}}     {{bars}}  {{count}} open   {{status}}
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:bulb: *Notable Today*
• {{insight 1 — pattern from data}}
• {{insight 2 — risk or anomaly}}
• {{insight 3 — coverage gap or positive note}}

:dart: *Actions*
1. *{{Person}}* → {{ticket}} {{account}}: {{what to do and why}}
2. ...
3. ...
```

## Rules

1. NEVER fabricate data. Only report what DevRev returns.
2. Clickable links: `<https://app.devrev.ai/shipsy/works/TKT-XXXXX|TKT-XXXXX>`
3. Cross-reference roster with ticket owners — flag if owner OFF.
4. Keep under 4000 characters.
5. Weekend: emphasize coverage gaps.
6. 3PL (4A+5A) is historically most stretched — always show SLA miss rate.
7. Exclude WMS team and WMS cohort.
8. Channel Signals: only show channels with actual activity. Mark others "Silent".
9. Notable Today: real insights from the data, not generic statements.
10. Actions: exactly 3-5 items, each with person + ticket + reason.
