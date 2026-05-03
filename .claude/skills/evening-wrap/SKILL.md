---
name: evening-wrap
description: Generate the Evening CX Wrap-up — resolved today, SLA performance, morning actions status, carry-over. Posts to C0A82U7MZ5F and C07BQD5776Y at 6 PM IST daily.
---

# Evening Wrap-up — CX Pulse

Post to TWO channels: C0A82U7MZ5F and C07BQD5776Y.

## Execution Steps

### Step 1: Pull Resolved Tickets Today
DevRev: search for tickets resolved/closed today. Get details via get_ticket.

### Step 2: Calculate Metrics
- Opened today vs resolved today (net change)
- SLA hit rate: hits / (hits + misses) * 100
- Avg resolution time in hours
- Benchmarks: FRT < 1 hour, resolution < 24 hours, SLA > 95%

### Step 3: Check Morning Actions
Read channel C0A82U7MZ5F for today's morning CX Pulse message. Extract ACTION items. Status: DONE / NOT DONE / IN PROGRESS.

### Step 4: Tomorrow's Roster
Google Sheet — tomorrow's column. Who's on duty.

### Step 5: Scan Channels Since Morning
- C07BQD5776Y — new frustrated alerts
- C0AU4MT6MUK — new escalations
- C033JL0BVN0 — war room activity

### Step 6: Format & Post

```
*CX Pulse — Evening Wrap-up* | {{day}}, {{date}}

*{{resolved}} resolved* · *{{opened}} opened* · *Net: {{+/-}}* · *SLA: {{pct}}%*
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:pushpin: *Scorecard*

```
Opened today:         {{count}}
Resolved today:       {{count}}  {{emoji}}
First response < 1hr: {{X}}/{{Y}} ({{%}})
Avg resolution time:  {{X}} hours
SLA hit rate:         {{%}}
```

> {{vs benchmark: "Above/below industry target"}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:white_check_mark: *Resolved Today*

```
Cohort         | Resolved | By Whom
---------------|----------|------------------
{{cohort}}     | {{cnt}}  | {{names}}
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:dart: *Morning Actions — Status*
1. {{action}} — *DONE* / *NOT DONE* / *IN PROGRESS*
...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:hourglass_flowing_sand: *Carrying Into Tomorrow*

```
Ticket         | Account     | Age  | Owner       | Risk
---------------|-------------|------|-------------|------
TKT-XXXXX      | {{account}} | {{d}}| {{owner}}   | {{color}}
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:busts_in_silhouette: *Tomorrow's Team*
> ON DUTY: {{names + shifts}}
> Top 3 priorities for tomorrow

:bulb: *Notable*
• {{insight about today's performance}}
```

## Rules

1. Never fabricate metrics.
2. Clickable ticket links.
3. Keep under 3000 characters. Shorter than morning.
4. Morning actions accountability is key — show DONE vs NOT DONE.
5. Skeleton crew days: "X-person team resolved Y tickets."
6. Exclude WMS team.
