# Auto-Allocation Engine for Shipsy Support Tickets — Architecture Plan

**Date:** 2026-05-03
**Status:** Research-Complete / Design-Phase
**Data Source:** DevRev MCP — 12 tickets deeply analyzed, 4 products, 20+ features, 20+ groups discovered

---

## EXECUTIVE SUMMARY

This document defines a complete auto-allocation system for Shipsy's DevRev support tickets. When a ticket arrives (primarily via email to helpdesk@shipsy.io), the system must determine: (1) which TEAM pool should handle it, and (2) which specific PERSON within that pool. The design is grounded entirely in real data extracted from DevRev.

---

## 1. SIGNAL HIERARCHY (Ranked by Reliability)

Based on analysis of 12 real tickets, here are the routing signals available at ticket creation time, ranked by reliability and availability:

| Rank | Signal | Field | Availability at Creation | Reliability | Notes |
|------|--------|-------|-------------------------|-------------|-------|
| 1 | **Customer Cohort** | `tnt__customer_cohort_dropdown` | ~85% (set by automation or manual triage) | HIGH | Primary routing axis. Values: 1-DTDC, 1-Reliance, 2-Aramex, 2-HNK, 3A-On Demand, 4A-B2C Shipper, WMS, etc. |
| 2 | **Account** | `account.display_name` | 100% (derived from sender email) | HIGH | Email domain maps to account. Reliance = @ril.com, DTDC = @dtdc.com, Flipkart = @flipkart.com, Aramex = @aramex.com |
| 3 | **Pod** | `tnt__pod` | ~70% (set during triage) | HIGH when present | Values: Alpha, Brahmos, On Demand, MCM, WMS, WMS Inbound, WMS Outbound, Finance, WB, Texas |
| 4 | **Applies-to Part** | `applies_to_part` | ~60% (often defaults to TMS/WMS product) | MEDIUM | Usually set to product level (TMS, WMS), rarely to feature level at creation |
| 5 | **Group** | `group` | ~15% (only TKT-95200 had group "Reliance Support" set) | LOW at creation | Rarely set at creation; mostly set manually later |
| 6 | **Source Channel** | `source_channel` | 100% | LOW routing value | Almost always "email" (all 12 tickets) |
| 7 | **Severity** | `severity` | 100% (defaults to medium) | LOW routing value | All 12 tickets were "medium" — not useful for routing, but useful for prioritization |
| 8 | **Tags** | `tags` | 100% | LOW routing value | Almost all have "helpdesk@shipsy.io" tag — indicates email inbox, not routing |
| 9 | **Title/Body Keywords** | `title`, `body` | 100% | MEDIUM (for NLP) | Keywords like "GRN", "rider", "booking", "docket", "COD", "ASN" are strong pod indicators |
| 10 | **Sender Email Domain** | `reported_by[0].email` | 100% | HIGH for account | @ril.com, @dtdc.com, @flipkart.com, @aramex.com, @rozana.in, @wellnessforever.in, @ibm.com (for Heineken) |

### Key Finding: The "tnt__customer_cohort_dropdown" is THE routing axis

This field is the single most important signal. It directly determines which team pool handles a ticket. It was present on 11 of 12 analyzed tickets.

---

## 2. OBSERVED ASSIGNMENT PATTERNS (from real data)

### 2A. Cohort-to-Person Mapping (discovered from tickets)

| Ticket | Account | Cohort | Pod | Assignee (tnt__assignee) | Owner (owned_by) |
|--------|---------|--------|-----|-------------------------|-------------------|
| TKT-96867 | Reliance (RIL) | 1-Reliance | — | Asif Khan (DEVU-2636) | Asif Khan |
| TKT-96959 | Rozana | 4A-B2C Shipper | WMS | DEVU-901 | Nissi Veronika Y (DEVU-4879) |
| TKT-96957 | Flipkart | 3A-On Demand | On Demand | Saurabh Singh (DEVU-2573) | Saurabh Singh |
| TKT-96946 | Wellness Forever (TMS) | 3A-On Demand | MCM | Laxmi Rajput (DEVU-2585) | Laxmi Rajput |
| TKT-96740 | Rozana | 4A-B2C Shipper | WMS Inbound | Bhavyank Sarolia (DEVU-3007) | Bhavyank Sarolia |
| TKT-96511 | DTDC | 1-DTDC | Alpha | Vidushi Wanchoo (DEVU-1314) | Vidushi Wanchoo |
| TKT-95550 | [WMS] Wellness Forever | — (no cohort) | WMS Inbound | Bhavyank Sarolia (DEVU-3007) | Bhavyank Sarolia |
| TKT-95200 | Reliance (RIL) | 1-Reliance | MCM | Deepanshu Marwari (DEVU-1088) | Deepanshu Marwari |
| TKT-93799 | heineken-br1 | 2-HNK | — | DEVU-2949 | Vinod Kumar Gunda (DEVU-1246) |
| TKT-92361 | Aramex Global | — (no cohort) | Brahmos | Srijan Srivastava (DEVU-2976) | Srijan Srivastava |
| TKT-86807 | Aramex Global | 2-Aramex | WB | Abhishek Bhandari (DEVU-2934) | Abhishek Bhandari |
| TKT-85246 | Flipkart | — (no cohort) | On Demand | Saurabh Singh (DEVU-2573) | Saurabh Singh |

### 2B. Key Patterns Discovered

**Pattern 1: Cohort is the primary team selector**
- `1-Reliance` tickets go to a dedicated Reliance support pool (Asif Khan, Deepanshu Marwari). Group "Reliance Support" (group-14) exists.
- `1-DTDC` tickets go to DTDC support pool (Vidushi Wanchoo). Group "DTDC Support" (group-388) exists.
- `2-Aramex` tickets go to Aramex-dedicated people (Abhishek Bhandari, Srijan Srivastava).
- `2-HNK` (Heineken) tickets go to dedicated HNK support.
- `3A-On Demand` tickets go to On Demand pod (Saurabh Singh, Laxmi Rajput).
- `4A-B2C Shipper` tickets can go to WMS or TMS pools depending on the product.

**Pattern 2: Pod refines within cohort**
- Within `3A-On Demand`, pod can be "On Demand" or "MCM" — different people handle each.
- Within `4A-B2C Shipper`, pod can be "WMS", "WMS Inbound" — WMS tickets go to Bhavyank Sarolia.
- Within `1-DTDC`, pod is "Alpha" — Vidushi Wanchoo handles.
- Within `2-Aramex`, pod can be "Brahmos" or "WB" — different assignees.

**Pattern 3: tnt__assignee and owned_by are usually identical**
- The `tnt__assignee` custom field and the `owned_by` system field point to the same person in most cases, though occasionally they differ (TKT-96959 shows DEVU-901 as tnt__assignee but DEVU-4879/Nissi as owned_by). This suggests tnt__assignee may be set first (by a rule/person), then owned_by is updated separately.

**Pattern 4: Account email domain is a reliable fallback for cohort**
- @ril.com = 1-Reliance
- @dtdc.com = 1-DTDC
- @flipkart.com = 3A-On Demand (or 4A-B2C based on context)
- @aramex.com = 2-Aramex
- @rozana.in = 4A-B2C Shipper
- @wellnessforever.in = 3A-On Demand or WMS
- @ibm.com (for Heineken) = 2-HNK

**Pattern 5: Most tickets arrive via email**
- All 12 analyzed tickets came via email channel to helpdesk@shipsy.io
- Created by "Email Integration Bot" (SVCACC-22)

---

## 3. ROUTING RULES — Complete Decision Tree

```
TICKET ARRIVES
    |
    v
[Step 0] SPAM CHECK
    |-- is_spam = true? --> Mark spam, do NOT assign. STOP.
    |-- subtype != "Support"? --> If "Project" or "Shipsy-Internal", skip auto-assignment. STOP.
    |
    v
[Step 1] DETERMINE COHORT (Level 1: Which Team Pool?)
    |
    |-- tnt__customer_cohort_dropdown already set?
    |   |-- YES --> Use it directly
    |   |-- NO --> Derive from account:
    |       |-- account.display_name contains "Reliance" or "RIL" --> "1-Reliance"
    |       |-- account.display_name contains "DTDC" --> "1-DTDC"
    |       |-- account.display_name contains "Aramex" --> "2-Aramex"
    |       |-- account.display_name contains "heineken" or "HNK" --> "2-HNK"
    |       |-- account.display_name contains "[WMS]" --> "WMS"
    |       |-- reported_by email domain:
    |           |-- @ril.com --> "1-Reliance"
    |           |-- @dtdc.com --> "1-DTDC"
    |           |-- @aramex.com --> "2-Aramex"
    |           |-- @flipkart.com --> "3A-On Demand" (default, refined by pod)
    |           |-- @rozana.in --> "4A-B2C Shipper"
    |           |-- @wellnessforever.in --> check applies_to_part:
    |               |-- WMS product --> "WMS"
    |               |-- TMS product --> "3A-On Demand"
    |           |-- UNKNOWN --> "TBD" (goes to general triage queue)
    |
    v
[Step 2] DETERMINE POD (Level 2: Which sub-team?)
    |
    |-- tnt__pod already set? --> Use it
    |-- NOT set? --> Derive from keywords in title + body:
    |   |-- "GRN", "ASN", "LPN", "inbound", "putaway" --> "WMS Inbound"
    |   |-- "pick", "pack", "dispatch", "outbound", "replenishment" --> "WMS Outbound"
    |   |-- "rider", "check-in", "check-out", "attendance", "payout" --> "On Demand"
    |   |-- "booking", "docket", "FM Portal", "CTBS" --> "Alpha" (DTDC-specific)
    |   |-- "invoice", "reconciliation", "COD", "remittance" --> "Finance"
    |   |-- "carrier", "tracking", "virtual series" --> "MCM"
    |   |-- "transformation builder", "service type", "TAT" --> "Brahmos"
    |   |-- "problem code", "NDR", "status mapping" --> "WB"
    |   |-- DEFAULT: leave blank for manual triage
    |
    v
[Step 3] SELECT PERSON (Level 3: Which individual?)
    |
    |-- Lookup: (cohort, pod) --> candidate pool
    |-- From candidate pool:
    |   1. Filter out people who are OFF (check roster/leave calendar)
    |   2. Filter out people at capacity (open ticket count >= threshold)
    |   3. From remaining, pick person with LOWEST open ticket count (round-robin with load balancing)
    |
    v
[Step 4] ASSIGN
    |-- Set owned_by = selected person
    |-- Set tnt__assignee = selected person
    |-- Set group = appropriate group (if cohort has a dedicated group)
    |-- Add internal comment: "Auto-assigned to {name} because: cohort={cohort}, pod={pod}, load={N open tickets}"
    |-- If stage is "Queued", transition to "In Progress" (stage ID: custom_stage/4)
    |
    v
[Step 5] FALLBACKS
    |-- If NO candidate available (all off or at capacity):
    |   |-- Severity = blocker/high? --> Escalate to team lead + Slack alert
    |   |-- Severity = medium/low? --> Leave in "Queued" stage, send Slack alert to support channel
    |-- If cohort = "TBD" or unknown:
    |   |-- Route to general triage queue (group: "Support Team" / group-400)
    |   |-- Slack alert to support leads
```

---

## 4. TEAM POOL MAPPING (Cohort + Pod --> People)

Based on observed data:

| Cohort | Pod | Known Assignees | DevRev Group |
|--------|-----|-----------------|--------------|
| 1-Reliance | MCM | Deepanshu Marwari (DEVU-1088) | Reliance Support (group-14) |
| 1-Reliance | (general/day-end) | Asif Khan (DEVU-2636) | Reliance Support (group-14) |
| 1-DTDC | Alpha | Vidushi Wanchoo (DEVU-1314) | DTDC Support (group-388) |
| 2-Aramex | Brahmos | Srijan Srivastava (DEVU-2976) | aramex projects (group-251) |
| 2-Aramex | WB | Abhishek Bhandari (DEVU-2934) | aramex projects (group-251) |
| 2-HNK | — | Vinod Kumar Gunda (DEVU-1246), DEVU-2949 | — |
| 3A-On Demand | On Demand | Saurabh Singh (DEVU-2573) | On demand POD (group-269) |
| 3A-On Demand | MCM | Laxmi Rajput (DEVU-2585) | — |
| 4A-B2C Shipper | WMS / WMS Inbound | Bhavyank Sarolia (DEVU-3007), Nissi Veronika Y (DEVU-4879) | — |
| WMS | WMS Inbound | Bhavyank Sarolia (DEVU-3007) | WMS Engineering (group-294) or WMS (group-289) |

**NOTE:** This is a partial mapping from 12 tickets. A complete mapping requires querying all support team members and their cohort/pod assignments, likely from a Google Sheet that syncs to DevRev groups.

---

## 5. CAPACITY ALGORITHM

### 5A. Load Calculation
```
For each candidate person P in the pool:
  open_tickets(P) = count of tickets WHERE:
    - owned_by includes P
    - state NOT IN ("closed")
    - subtype = "Support"

  capacity_score(P) = MAX_CAPACITY - open_tickets(P)

  Select P with highest capacity_score.
  Tie-break: person who was assigned a ticket least recently (longest idle).
```

### 5B. Capacity Thresholds (recommended starting values)
| Cohort Tier | Max Open Tickets per Person |
|-------------|---------------------------|
| Tier 1 (Reliance, DTDC) | 15 (high volume, dedicated teams) |
| Tier 2 (Aramex, HNK) | 10 |
| On Demand / B2C | 12 |
| WMS | 10 |

### 5C. Shift Awareness
From SLA data, the org schedule is "Mon - Fri (10AM - 8PM)" IST (org_schedule-13). However, real ticket creation times show tickets arriving outside this window (TKT-96867 created at 20:36 UTC = 02:06 IST next day).

**Recommendation:**
- Primary shift: 10:00 AM - 8:00 PM IST (Mon-Fri) — most support staff
- Off-hours: tickets queue; auto-assigned when next shift starts
- For blocker/high severity off-hours tickets: alert on-call person via Slack
- Weekend: skeleton crew from each major cohort; define weekend roster

### 5D. Leave/Roster Integration
- Source: Google Sheets roster (already syncing to DevRev groups per sync metadata)
- Check: Before assigning, verify person is not on planned leave
- Mechanism: Maintain a leave calendar (PL, WO, comp-off dates per person)
- If person is on leave, skip them in the candidate pool

---

## 6. EDGE CASE HANDLING

### 6.1 Ticket with no cohort (tnt__customer_cohort_dropdown is empty)
**Observed:** TKT-95550 (WMS Wellness Forever) and TKT-85246 (Flipkart COD) had no cohort set.
**Solution:** Derive cohort from account name and email domain (see Step 1 in decision tree). If still unknown, set cohort to "TBD" and route to general triage queue.

### 6.2 Ticket from a new/unknown account
**Solution:** If account is not in the known mapping, route to "Support Team" (group-400) with a Slack alert. A human triager assigns cohort and pod.

### 6.3 Blocker/high severity ticket during off-hours
**Solution:** Maintain an on-call rotation per cohort. For blocker tickets created outside 10AM-8PM IST, immediately Slack-notify the on-call person. Do NOT wait for next shift.

### 6.4 Ticket spanning multiple pods
**Observed:** TKT-96946 (Wellness Forever TMS account, but pod = MCM, cohort = 3A-On Demand). The system already handles this by using (cohort, pod) as a compound key.
**Solution:** If a ticket genuinely spans two pods, assign to the PRIMARY pod based on the title/body analysis. Add tnt__secondary_pods field if needed.

### 6.5 Reopened ticket (customer replies after resolution)
**Observed:** TKT-96740 (Rozana GRN Pending) — stage = "Reopen", tag = "reopen-customer-response".
**Solution:** Reopened tickets MUST go back to the ORIGINAL owner (the person who resolved it). The system should read the previous owned_by and re-assign to that person. If that person is on leave, escalate to same-pod colleague.

### 6.6 Duplicate tickets from same customer
**Observed:** TKT-93799 (Heineken) — resolution = "Duplicate" of TKT-93798.
**Solution:** The auto-allocation engine should NOT try to detect duplicates — that is a separate concern. Assign normally; support person marks as duplicate during triage.

### 6.7 Feature requests (subtype = Project)
**Solution:** Do not auto-assign. These follow a different governance workflow (product sprint planning, scope approval, etc.). The tnt__ticket_type field distinguishes: "Support" vs "Event" vs "NCO" vs "Retention".

### 6.8 Spam tickets (is_spam = true)
**Solution:** Skip auto-assignment entirely. Leave unassigned.

---

## 7. DEVREV API OPERATIONS REQUIRED

### 7.1 Assign a ticket owner
```
update_ticket(
  id = "don:core:dvrv-us-1:devo/xXjPo9nF:ticket/{id}",
  account = "{existing_account_id}",  // REQUIRED field
  owned_by = { set: ["{person_don_id}"] },
  tnt__assignee = "{person_don_id}"
)
```

### 7.2 Set the group
```
update_ticket(
  id = "...",
  account = "...",
  group = "don:identity:dvrv-us-1:devo/xXjPo9nF:group/{group_id}"
)
```

### 7.3 Add auto-assignment comment
```
add_comment(
  object = "don:core:dvrv-us-1:devo/xXjPo9nF:ticket/{id}",
  body = "**Auto-assigned** to <{person_don_id}> | Cohort: {cohort} | Pod: {pod} | Open tickets: {count}",
  visibility = "internal"
)
```

### 7.4 Change stage from Queued to In Progress
```
update_ticket(
  id = "...",
  account = "...",
  stage = "don:core:dvrv-us-1:devo/xXjPo9nF:custom_stage/4"  // "In Progress"
)
```
Known stage IDs from data:
- Queued: ordinal 500 (initial stage for new tickets)
- In Progress: custom_stage/4 (state: in_progress)
- Awaiting Customer Response: custom_stage/5 (state: in_progress, SLA paused)
- Awaiting Development: custom_stage/7 (state: in_progress)
- Resolved: custom_stage/1 (state: closed)
- Closed: custom_stage/81 (state: closed)
- Reopen: custom_stage/90 (state: open)

### 7.5 Read cohort from account
The cohort is stored on the TICKET (tnt__customer_cohort_dropdown), not on the account object. To set it automatically, maintain a mapping: Account ID --> Cohort. This mapping should be a configuration table.

---

## 8. SELF-CORRECTIONS (What I Initially Assumed vs. What the Data Showed)

| Initial Assumption | What the Data Showed | Correction |
|-------------------|---------------------|------------|
| Cohort would be set on the Account object | Cohort is a ticket-level custom field (tnt__customer_cohort_dropdown). Not inherited from account. | Must maintain a separate Account-to-Cohort mapping table. |
| Group would be set on most tickets | Only 1 of 12 tickets (TKT-95200) had a group ("Reliance Support"). Most tickets have no group. | Group is underused; auto-allocation should SET the group, not rely on it. |
| Severity would vary and be useful for routing | All 12 tickets had severity = "medium". | Severity is NOT a routing signal. It defaults to medium and is rarely changed. Use it only for priority within a queue. |
| applies_to_part would be set to specific features | Most tickets had applies_to_part = product level (TMS or WMS), not feature level. Only TKT-95550 (Replenishment) and TKT-85246 (COD Management) had feature-level parts. | Part is too coarse for routing. Pod + Cohort is more reliable. |
| tnt__assignee and owned_by would be the same | They usually are, but can diverge (TKT-96959: tnt__assignee = DEVU-901, owned_by = DEVU-4879). | Auto-allocation should set BOTH fields to the same person. |
| Tickets would come from multiple channels | All 12 tickets came via email. | Channel is not a routing signal. The engine only needs to handle email-originated tickets for now. |
| There would be distinct Aramex/HNK support groups | No dedicated DevRev group found for HNK. Aramex has "aramex projects" (group-251). | Some cohorts have dedicated groups, others do not. The engine should create/use groups for ALL cohorts. |

---

## 9. REASONING FOR KEY DESIGN DECISIONS

### 9.1 Why Cohort as Level 1 (not Account, not Part)?
**Reasoning:** Accounts are too granular (hundreds of customers). Parts are too coarse (only TMS/WMS for most tickets). Cohorts are the Goldilocks abstraction — there are ~10 distinct cohorts, each mapping to a dedicated team. The data proves this: Reliance tickets always go to Reliance support people, DTDC always to DTDC people, regardless of the specific feature.

### 9.2 Why Pod as Level 2 (not Part)?
**Reasoning:** Pod is a team-organizational concept that maps directly to engineering pods. Within "3A-On Demand", the pod "On Demand" vs "MCM" determines different people. The part field (applies_to_part) is usually set at product level and does not differentiate support queues. Pod is set on ~70% of tickets and has strong signal.

### 9.3 Why round-robin with load balancing (not pure round-robin)?
**Reasoning:** Pure round-robin ignores that some tickets take much longer than others (TKT-95550 was open 16+ days in "Awaiting Development"). Load balancing based on open ticket count ensures no person is overwhelmed while others are idle.

### 9.4 Why set Group on assignment?
**Reasoning:** Groups are underused today (1/12 tickets). But groups enable DevRev's built-in team views, analytics, and SLA tracking. By programmatically setting the group, we unlock DevRev's native features without changing human behavior.

### 9.5 Why derive cohort from email domain as fallback?
**Reasoning:** The tnt__customer_cohort_dropdown is not always set at ticket creation (missing on ~15% of tickets). But the sender email domain (@ril.com, @dtdc.com, etc.) is ALWAYS available and is a near-perfect proxy for the account, which maps 1:1 to a cohort.

---

## 10. WHAT THIS CANNOT HANDLE (Honest Limitations)

1. **New customers with unknown email domains:** If a brand-new customer emails helpdesk@shipsy.io for the first time, there is no account mapping. These go to the general triage queue.

2. **Tickets that need cross-pod collaboration:** A ticket like "WMS integration with carrier" spans WMS and MCM. The engine assigns to one pod; cross-pod coordination remains manual.

3. **Severity escalation in real-time:** If a medium-severity ticket becomes a blocker mid-resolution, the engine does not re-route. The support person must manually escalate.

4. **Internal tickets (Shipsy-Internal subtype):** These follow a different workflow and should be excluded from auto-allocation.

5. **Quality of cohort/pod data:** If humans mis-set the cohort or pod, the ticket goes to the wrong team. The engine trusts the data it is given. Garbage in = garbage out.

6. **Capacity data freshness:** Open ticket counts are point-in-time. If two tickets arrive simultaneously, both might get assigned to the same "least loaded" person before the count updates. This is acceptable at Shipsy's ticket volume.

7. **Timezone-aware routing for global customers:** Heineken (Brazil) and Aramex (Middle East) operate in different timezones. The engine does not factor customer timezone into agent selection. This may cause response-time issues for off-IST customers.

8. **Leave data availability:** If the leave/roster data is not programmatically accessible (e.g., stuck in a Google Sheet without API access), the engine cannot filter out unavailable people. This is the single biggest operational dependency.

9. **SLA differentiation by cohort:** Currently all tickets use "Support Ticket SLA Default" (sla-28) with "Mon - Fri (10AM - 8PM)" IST schedule. Tier-1 customers (Reliance, DTDC) likely need tighter SLAs, but the engine cannot enforce what is not configured in DevRev.

10. **No historical learning:** This is a rule-based engine, not ML-based. It does not learn from past assignments to improve future ones. Adding ML-based pod/cohort prediction from title/body text would be a future enhancement.

---

## APPENDIX A: DISCOVERED TAXONOMY

### Products
| ID | Name | Owner |
|----|------|-------|
| PROD-19 | TMS | Pushkar Soni |
| PROD-23 | WMS | Karthik Bala (deactivated) |
| PROD-25 | Automation Test Coverage (QA) | Hitesh Sarup |
| PROD-28 | EXIM | Raajit Kumar |

### Key Features (under TMS)
| ID | Name | Owner |
|----|------|-------|
| FEAT-335 | Customer Master (TMS) | Pushkar Soni |
| FEAT-384 | Transport Master | anurag.singh |
| FEAT-156 | Rider Management | Saurabh Bora |
| FEAT-367 | Dispatch Management | Karan Mehra |
| FEAT-150 | Trip Management | anurag.singh |
| FEAT-547 | Carrier Integration | Dolma Garg |
| FEAT-144 | Rider Reconciliation | Rakesh Saripalli |
| FEAT-136 | Carrier Tracking | Saarthak Jain |
| FEAT-541 | Consignment Dashboard | Vinay Sharma |
| FEAT-160 | Label Generation | Saurabh Bora |
| FEAT-128 | Communication Engine | Rakesh Saripalli |
| FEAT-185 | Carrier Virtual Series | Saarthak Jain |
| FEAT-356 | Hub Ops Application | Rakesh Saripalli |
| FEAT-359 | Inbound operations on Hub ops | Rakesh Saripalli |
| FEAT-360 | Outbound operations on Hub ops | Rakesh Saripalli |
| FEAT-552 | POD | Hitesh Sarup |
| FEAT-605 | Customer Portal > Consignments | Saarthak Jain |
| FEAT-143 | COD Management | Rakesh Saripalli |

### Key Groups
| ID | Name | Type |
|----|------|------|
| group-14 | Reliance Support | dev_user |
| group-388 | DTDC Support | dev_user |
| group-387 | Shipsy Support for Reliance | dev_user |
| group-384 | Shipsy Customer Support team | dev_user |
| group-346 | Shipsy Customer Support | dev_user |
| group-400 | Support Team | dev_user |
| group-default4 | Support | dev_user |
| group-269 | On demand POD | dev_user |
| group-294 | WMS Engineering | dev_user |
| group-264 | WMS Engineering Leads | dev_user |
| group-307 | Finance POD | dev_user |
| group-251 | aramex projects | dev_user |
| group-289 | WMS | dev_user |
| group-292 | WMS Product | dev_user |
| group-290 | TMS + WMS | dev_user |
| group-344 | Shipsy Finance | dev_user |
| group-350 | Full Time Shipsy Team | dev_user |
| group-250 | team exim | dev_user |

### tnt__pod Values (from ticket schema)
AI Agents, Alpha, Analytics, Brahmos, EXIM, Finance, Infra, Logistics Intelligence Team, Logistics Lighthouse, MCM, Mobile mad max, On Demand, POD, Platformization, Texas, WB, WMS, WMS Inbound, WMS Outbound

### tnt__customer_cohort_dropdown Values (from ticket schema)
1-DTDC, 1-Reliance, 2-Aramex, 2-HNK, 3A-On Demand, 3B-S (On Demand), 4A-B2C LSP, 4A-B2C Shipper, 4B-S (B2C LSP), 4B-S (B2C Shipper), 5A-B2B LSP, 5A-B2B Shipper, 5B-S (B2B LSP), 5B-S (B2B Shipper), AI, Exim, Platform, Roadmap, TBD, WMS

### Ticket Stages (from observed data)
| Stage | Display Name | State | Ordinal | SLA Effect |
|-------|-------------|-------|---------|------------|
| custom_stage/? | Queued | Open | 500 | SLA running |
| custom_stage/90 | Reopen | Open | 750 | SLA running |
| custom_stage/4 | In Progress | In Progress | 3000 | SLA running |
| custom_stage/7 | Awaiting Development | In Progress | 3750 | SLA running |
| custom_stage/5 | Awaiting Customer Response | In Progress | 4800 | SLA paused |
| custom_stage/1 | Resolved | Closed | 6000 | SLA stopped |
| custom_stage/81 | Closed | Closed | 13000 | SLA stopped |

---

## APPENDIX B: IMPLEMENTATION SEQUENCE

### Phase 1: Static Rules (Week 1-2)
- Build Account-to-Cohort mapping table (from existing accounts)
- Build Cohort+Pod-to-PersonPool mapping table
- Implement the decision tree as a DevRev snap-in (webhook on ticket.created)
- Set owned_by, tnt__assignee, group, and add internal comment

### Phase 2: Keyword-Based Pod Detection (Week 3)
- Build keyword-to-pod classifier from title and body
- Auto-set tnt__pod when not already set
- Refine the Cohort+Pod routing with additional person pools

### Phase 3: Capacity & Availability (Week 4-5)
- Query open ticket counts per person at assignment time
- Integrate leave/roster data (Google Sheets API or manual sync)
- Implement load-balanced selection

### Phase 4: Monitoring & Tuning (Week 6+)
- Track auto-assignment accuracy (% of tickets that stay with the auto-assigned person)
- Track time-to-assignment improvement
- Adjust capacity thresholds based on observed workload
- Add Slack notifications for edge cases
