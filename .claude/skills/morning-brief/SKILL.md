---
name: morning-brief
description: Generate the daily Morning CX Brief — pulls DevRev tickets, roster from Google Sheet, Slack channel signals, and posts structured report to leadership Slack channel. Trigger at 8 AM IST daily.
---

# Morning Brief — CX Pulse

You generate the Morning CX Brief for Shipsy support leadership. Post to Slack channel C0A82U7MZ5F.

## Execution Steps

### Step 1: Read Roster
Read Google Sheet `1v8lbH2yZCU7TAInUNO2tqx-HDGbCjPVtqZEWX94pApc`.
Find today's date column. Extract:
- ON DUTY: name + shift time
- OFF TODAY: names on WO (Week Off) or PL (Planned Leave)

### Step 2: Pull DevRev Tickets
DevRev is the source of truth. Use DevRev MCP tools.

**Searches to run:**
- `hybrid_search namespace=ticket query="all open support tickets not resolved not closed" limit=50`
- `hybrid_search namespace=ticket query="support tickets created in last 24 hours" limit=50`
- `hybrid_search namespace=ticket query="unassigned support tickets" limit=50`

**For each ticket, get via `get_ticket`:**
- `display_id`, `title`, `body` (first 100 chars)
- `tnt__customer_cohort_dropdown` — THE cohort (see rules/cohort-mapping.md)
- `tnt__pod` — sub-team
- `severity`, `severity_v2.label`
- `stage.name`, `state`
- `owned_by[0].display_name` — current owner
- `account.display_name` — customer
- `sentiment.label`, `sentiment_summary`
- `sla_summary` — hit or miss
- `needs_response` — true/false
- `created_date`

**Group by cohort:**
- DEDICATED: 1-Reliance, 1-DTDC (Tier 1), 2-Aramex, 2-HNK (Tier 2)
- 3PL: 4A-B2C Shipper, 4A-B2C LSP, 5A-B2B LSP, 5A-B2B Shipper, 4B-S, 5B-S
- ON-DEMAND: 3A-On Demand, 3B-S (On Demand)
- WMS: WMS
- OTHER: Exim, Platform, Roadmap, TBD, AI, empty/null

### Step 3: Scan Slack Channels (last 24h)
Read with `oldest` = 24 hours ago, `limit` = 20, `response_format` = concise:

**Tier 1 — Frustrated alerts + escalations:**
- `C07BQD5776Y` (#customer-experience-product-support)
- `C0AU4MT6MUK` (#support-urgent-help)
- `C07KDUT9D5M` (#support-blocker-p1-tickets)

**Tier 2 — WhatsApp bridges:**
- `C09P8BC41PW` (#shipsy-aramex-whatsapp)
- `C0ACT5ER2E6` (#chronodiali-shipsy-whatsapp)
- `C09L2ARTF4J` (#flipkart-shipsy-operation-check-whatsapp)
- `C09EDETE468` (#shipsy-ril-escalation-whatsapp)

**Tier 3 — External customer channels:**
- `C081FG99KKL` (#ext-starlinks-shipsy-support)
- `C081GJ2M0LW` (#ext-qatarpost-qp-shipsy)
- `C082A7CUDFU` (#ext-chronodiali-shipsy)
- `C09TGCAKNK1` (#ext-healthkart-shipsy-support)
- `C07UZFTU7C4` (#movin)

Extract: ticket IDs mentioned, customer complaints, urgent issues, Periskope Bot messages.

### Step 4: Check GitHub Deploys (last 24h)
If GitHub MCP available, check for:
- Production deploys (merges to `release` branch)
- Mobile build tags (v*.*.*)
Only include if found. Skip silently if no deploys.

### Step 5: Format & Post

Post to `C0A82U7MZ5F` using `slack_send_message`:

```
*CX Pulse — Daily Support Brief*
_{{day}}, {{date}}_

---

*TODAY'S TEAM* _(from roster)_
ON DUTY: {{name — shift}} per line
OFF TODAY: {{names}}
> {{coverage summary: "X people covering all accounts" or "Skeleton crew — Y people only"}}

---

*OPEN TICKETS — by Customer Cohort*

*DEDICATED — 1-Reliance / 1-DTDC*
> *{{Account}}* — {{count}} open
> {{Owner}} ({{count}}): {{ticket title one-liner}} — {{if owner OFF today, flag it}}
> _SLA: {{hit/miss}}. Sentiment: {{label}}._

*DEDICATED — 2-Aramex / 2-HNK*
> [same format]

*3PL SECTION* _(4A + 5A — highest load section)_
> *{{Callout if SLA miss rate high or understaffed}}*
> *{{Account}}* — {{count}} open
> {{Owner or UNASSIGNED}}: {{title}} — {{sentiment if frustrated/unhappy}}
> *3PL workload: {{X}} tickets, {{Y}} orgs, {{Z}} people*

*ON-DEMAND — 3A*
> [same format]

*WMS*
> [same format, if any WMS tickets open]

*OTHER / UNTAGGED*
> [tickets without cohort]

---

*CHANNEL SIGNALS* _(last 24h — only if signals exist)_
> #customer-experience-product-support: {{count}} frustrated alerts
> Each: TKT-XXXXX {{account}} — owner: {{name}} (on/off today)
> #support-urgent-help: {{escalation one-liners}}
> WhatsApp/External: {{customer issues outside DevRev}}

---

*SHIPPED YESTERDAY* _(only if deploys found)_
> {{deploy description}} — merged by {{author}}

---

*WORKLOAD*
```
ON DUTY:   {{Name}}  {{bars}}  {{count}} open  {{shift}}  ({{cohort}})
OFF TODAY: {{Name}}  {{bars}}  {{count}} open  {{status}}
```

---

*TODAY'S ACTIONS* _(3-5 items)_
1. {{Person}} — {{action}} — {{ticket}} — {{why}}
...

---
_CX Pulse — DevRev + Roster + Slack + GitHub_
_Reply in thread to suggest changes_
```

## Rules

1. NEVER fabricate data. If DevRev query fails, say "DevRev data unavailable."
2. Every ticket ID = clickable link: `<https://app.devrev.ai/shipsy/works/TKT-XXXXX|TKT-XXXXX>`
3. Cross-reference roster with ticket owners — flag if owner is OFF today.
4. Keep under 4000 characters.
5. On weekends, emphasize coverage gaps.
6. 3PL section (4A+5A) is historically the most stretched — always show its workload ratio.
7. CHANNEL SIGNALS only if real messages found. No empty sections.
8. SHIPPED YESTERDAY only if actual deploys found. No empty sections.
9. Actions must name a person + ticket + reason. No vague items.
10. Tone: professional, concise, data-driven. No emoji. No filler.
