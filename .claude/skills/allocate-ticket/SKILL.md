---
name: allocate-ticket
description: Auto-allocate a support ticket to the right person. Reads cohort, pod, roster, and capacity. Assigns via DevRev API and notifies via Slack DM. Use when a new support ticket arrives or when manually triggered for reallocation.
---

# Auto-Allocate Ticket — CX Pulse

When a support ticket arrives, determine the RIGHT person and assign it.

## Decision Tree

### Step 0: Pre-checks
- `is_spam = true` -> SKIP. Do not assign.
- `subtype != "Support"` -> SKIP. Projects/Internal follow different workflow.
- Already has `owned_by` != "Unassigned" -> SKIP. Already assigned.

### Step 1: Determine Cohort (which team pool?)

Read `tnt__customer_cohort_dropdown` from the ticket.

**If set:** use directly. See rules/cohort-mapping.md for the full list.

**If empty:** derive from available signals:
1. `account.display_name` contains "Reliance" or "RIL" -> `1-Reliance`
2. `account.display_name` contains "DTDC" -> `1-DTDC`
3. `account.display_name` contains "Aramex" -> `2-Aramex`
4. `account.display_name` contains "heineken" or "HNK" -> `2-HNK`
5. `account.display_name` contains "[WMS]" -> `WMS`
6. `reported_by[0].email` domain:
   - @ril.com -> `1-Reliance`
   - @dtdc.com -> `1-DTDC`
   - @aramex.com -> `2-Aramex`
   - @flipkart.com -> `3A-On Demand`
   - @rozana.in -> `4A-B2C Shipper`
   - @wellnessforever.in -> check applies_to_part (WMS vs TMS)
7. UNKNOWN -> `TBD` -> general triage queue

**WHY cohort first:** Accounts are too granular (hundreds). Parts are too coarse (TMS/WMS). Cohorts are the Goldilocks abstraction — ~20 values, each mapping to a dedicated team. Data proves: Reliance tickets always go to Reliance people, regardless of feature.

### Step 2: Determine Pod (which sub-team?)

Read `tnt__pod` from the ticket.

**If set:** use directly.

**If empty:** keyword scan `title` + `body`:
- "GRN", "ASN", "LPN", "inbound", "putaway" -> `WMS Inbound`
- "pick", "pack", "dispatch", "outbound" -> `WMS Outbound`
- "rider", "check-in", "payout", "attendance" -> `On Demand`
- "booking", "docket", "FM Portal", "CTBS" -> `Alpha`
- "invoice", "reconciliation", "COD", "remittance" -> `Finance`
- "carrier", "tracking", "virtual series" -> `MCM`
- "transformation builder", "service type", "TAT" -> `Brahmos`
- "problem code", "NDR", "status mapping" -> `WB`
- No match -> leave blank for manual triage

**WHY pod second:** Pod maps to engineering team structure. Within "3A-On Demand", pod "On Demand" vs "MCM" routes to different people. Part field (applies_to_part) is usually set at product level — not granular enough.

### Step 3: Select Person

Lookup `(cohort, pod)` in rules/team-pools.md -> candidate pool.

**Selection algorithm:**
1. Filter out people who are OFF today (check roster)
2. Filter out people at capacity (open ticket count >= threshold from config/thresholds.json)
3. From remaining, pick person with LOWEST open ticket count
4. Tie-break: person who was assigned least recently (longest idle)

**WHY load-balanced round-robin:** Pure round-robin ignores that some tickets take days. Load balancing by open count ensures no one is overwhelmed while others are idle.

### Step 4: Execute Assignment

```
update_ticket(
  id = ticket_don_id,
  owned_by = { set: [person_don_id] }
)
```

Also set:
- `tnt__assignee` = person_don_id
- `group` = cohort's DevRev group (see rules/team-pools.md)
- Stage: if "Queued", transition to "In Progress" (custom_stage/4)
- Add internal comment: "Auto-assigned to {name} | Cohort: {cohort} | Pod: {pod} | Open tickets: {count}"

### Step 5: Notify

Slack DM to assignee:
> New ticket assigned to you: <ticket_link|TKT-XXXXX>
> Account: {account} | Cohort: {cohort} | Severity: {severity}
> Title: {title}

### Step 6: Fallbacks

- **All candidates OFF or at capacity:**
  - Blocker/high -> escalate to team lead + Slack alert to channel
  - Medium/low -> leave in Queued, Slack alert to support channel
- **Cohort = TBD or unknown:**
  - Route to "Support Team" (group-400)
  - Slack alert to support leads
- **Reopened ticket (stage = Reopen):**
  - Re-assign to ORIGINAL owner (the person who resolved it)
  - If original owner on leave -> same-pod colleague

## Edge Cases

1. No cohort + unknown email domain -> general triage queue
2. Multi-pod ticket -> assign to PRIMARY pod from title analysis
3. Blocker off-hours -> on-call person via Slack
4. Duplicate tickets -> assign normally; support person marks duplicate
5. Feature requests (subtype=Project) -> skip auto-allocation
6. Spam (is_spam=true) -> skip entirely
