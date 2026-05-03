---
name: evening-wrap
description: Generate the Evening CX Wrap-up — compares against morning baseline, shows which tickets resolved vs still open, SLA performance, new tickets. Posts to C0A82U7MZ5F and C07BQD5776Y at 6 PM IST daily.
---

# Evening Wrap-up — CX Pulse

Post to TWO channels: C0A82U7MZ5F and C07BQD5776Y.

## Core Concept: Morning → Evening Closed Loop

The morning brief lists all open tickets as the "Morning Baseline."
The evening wrap-up re-checks EVERY ticket from the morning list and reports:
- Which got **RESOLVED** during the day
- Which are **STILL OPEN**
- What **NEW tickets** arrived today

This creates accountability: leadership sees exactly what happened during the day.

## Execution Steps

### Step 1: Read Morning Baseline
Read channel C0A82U7MZ5F for today's morning CX Pulse message (posted at ~8 AM).
Extract ALL ticket IDs (TKT-XXXXX) from the "Open Tickets" section.
This is the morning baseline list.

### Step 2: Re-check Each Morning Ticket
For EACH ticket ID from the morning baseline, call `get_ticket`:
- If `state` = "closed" → mark as **RESOLVED TODAY**. Note who resolved it (`owned_by`), resolution type (`tnt__resolution`), and duration.
- If `state` = "open" or "in_progress" → mark as **STILL OPEN**. Note current stage and any change from morning.

### Step 3: Find NEW Tickets Created Today
Search DevRev for tickets created today:
- `hybrid_search namespace=ticket query="support tickets created today" limit=50`
- For each, get_ticket. Filter: subtype=Support, created today.
- These are tickets NOT in the morning baseline — they arrived during the day.

### Step 4: Calculate Metrics
- Morning baseline count
- Resolved today (from baseline): count + who resolved
- Still open (from baseline): count
- New today: count
- Net change: (morning open + new) - resolved = evening open
- SLA hit rate: for resolved tickets, count hit vs miss
- Avg resolution time: for resolved tickets, avg(close_date - created_date) in hours
- Industry benchmarks: FRT < 1 hour, resolution < 24 hours, SLA > 95%

### Step 5: Check Morning Actions Status
Read the morning message's "Actions" section. For each action:
- Check if the ticket mentioned was resolved → DONE
- Check if stage changed → IN PROGRESS
- No change → NOT DONE

### Step 6: Tomorrow's Roster
Google Sheet — tomorrow's column. Who's on duty.

### Step 7: Scan Channels Since Morning
- C07BQD5776Y — new frustrated alerts since 8 AM
- C0AU4MT6MUK — new escalations since 8 AM
- C033JL0BVN0 — war room activity today

### Step 8: Format & Post to BOTH channels

```
*CX Pulse — Evening Wrap-up* | {{day}}, {{date}}

*{{resolved}} resolved* · *{{new}} new* · *{{still_open}} still open* · *Net: {{+/-}}* · *SLA: {{pct}}%*
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:pushpin: *Scorecard*

```
Morning baseline:     {{count}} open tickets
Resolved today:       {{count}}
New tickets today:    {{count}}
Still open tonight:   {{count}}
Net change:           {{+/-}}
SLA hit rate:         {{%}}
Avg resolution time:  {{X}} hours
```

> {{vs benchmark: "Above/below industry target"}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:arrows_counterclockwise: *Morning Baseline → Evening Status*

```
Ticket    | Account          | Morning          | Evening
----------|------------------|------------------|------------------
TKT-XXXXX | {{account}}      | {{morning stage}} | RESOLVED by {{name}}
TKT-XXXXX | {{account}}      | {{morning stage}} | STILL OPEN - {{current stage}}
...
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:new: *New Tickets Today* (not in morning baseline)

• TKT-XXXXX {{Account}} — {{title}} | {{owner}} | {{severity}}
...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:dart: *Morning Actions — Status*
1. {{action}} — *DONE* / *NOT DONE* / *IN PROGRESS*
...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:satellite: *Channel Signals Since Morning*
> {{New frustrated alerts, escalations, incidents since 8 AM}}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

:busts_in_silhouette: *Tomorrow's Team*
> ON DUTY: {{names + shifts}}

:bulb: *Tomorrow's Priorities*
1. {{carry-over ticket that needs attention}}
2. {{new ticket from today that's unresolved}}
3. {{any escalation pending}}
```

## Rules

1. Never fabricate metrics. Only report what DevRev returns.
2. Clickable ticket links.
3. The Morning Baseline → Evening Status table is THE core section. Every morning ticket MUST appear with a clear RESOLVED or STILL OPEN status.
4. Keep under 3500 characters.
5. Morning actions accountability: show DONE vs NOT DONE for every action item.
6. Skeleton crew days: "X-person team resolved Y tickets."
7. Exclude WMS team.
8. If morning message not found in channel, say "Morning baseline unavailable" and report current open tickets instead.
9. Tone: celebratory when net positive ("strong day"), factual when not.
