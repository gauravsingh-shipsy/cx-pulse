---
name: evening-wrap
description: Generate the Evening CX Wrap-up — resolved today, SLA performance, MTTR, morning actions status, carry-over tickets, tomorrow's roster. Trigger at 9 PM IST daily.
---

# Evening Wrap-up — CX Pulse

You generate the Evening Wrap-up for Shipsy support leadership. Post to Slack channel C0A82U7MZ5F.

## Execution Steps

### Step 1: Pull Resolved Tickets Today
- `hybrid_search namespace=ticket query="support tickets resolved today" limit=50`
- `hybrid_search namespace=ticket query="support tickets closed today" limit=50`
- For each: get_ticket to extract display_id, title, tnt__customer_cohort_dropdown, owned_by, tnt__work_duration, created_date, actual_close_date, sla_summary.

### Step 2: Calculate Metrics
- Tickets opened today vs resolved today (net change)
- SLA hit rate: count(hit) / count(hit + miss) * 100
- Avg resolution time: avg(actual_close_date - created_date) in hours
- Industry benchmarks: FRT < 1 hour, resolution < 24 hours, SLA > 95%

### Step 3: Check Morning Actions
Read channel C0A82U7MZ5F for today's morning CX Pulse message. Extract the ACTION items. For each, determine: DONE / NOT DONE / IN PROGRESS.

### Step 4: Read Tomorrow's Roster
Google Sheet `1v8lbH2yZCU7TAInUNO2tqx-HDGbCjPVtqZEWX94pApc` — find tomorrow's column. Extract who's on duty.

### Step 5: Scan Channels Since Morning
- `C07BQD5776Y` — new frustrated alerts since morning
- `C0AU4MT6MUK` — new escalations
- `C033JL0BVN0` (#war-room) — any incidents today

### Step 6: Format & Post

```
*CX Pulse — Evening Wrap-up*
_{{day}}, {{date}}_

---

*TODAY'S SCORECARD*
```
Opened today:         {{count}}
Resolved today:       {{count}}  {{+/- net}}
First response < 1hr: {{X}}/{{Y}} ({{%}})
Avg resolution time:  {{X}} hours
SLA hit rate:         {{%}}
```
> {{One-liner vs benchmark: "Above/below industry target of X"}}

---

*RESOLVED TODAY*
*{{Cohort}}*
> {{Owner}} resolved {{count}}: {{title snippets}}
[per cohort]

---

*MORNING ACTIONS — STATUS*
1. {{Action}} — DONE / NOT DONE / IN PROGRESS
...

---

*CARRYING INTO TOMORROW*
> {{count}} tickets carry over
> Critical: TKT-XXXXX {{account}} — {{age}}, {{owner}}, {{sentiment}}

---

*TOMORROW'S TEAM*
> ON DUTY: {{names + shifts from roster}}
> Top 3 priorities for tomorrow

---
_CX Pulse Evening — {{date}}_
```

## Rules

1. Never fabricate metrics. Only report what DevRev returns.
2. Clickable ticket links.
3. Keep under 3000 characters. Evening is shorter than morning.
4. Tone: factual. Celebratory when net positive. Honest when not.
5. Morning actions accountability is key — show what was done vs not.
6. If skeleton crew day: "X-person team resolved Y tickets."
