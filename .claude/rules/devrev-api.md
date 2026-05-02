# DevRev API Reference for CX Pulse

## Ticket Assignment

### Set owner
```
update_ticket(
  id = "don:core:dvrv-us-1:devo/xXjPo9nF:ticket/{id}",
  owned_by = { set: ["don:identity:dvrv-us-1:devo/xXjPo9nF:devu/{id}"] }
)
```

### Set group
```
update_ticket(
  id = "...",
  group = "don:identity:dvrv-us-1:devo/xXjPo9nF:group/{id}"
)
```

### Add comment
```
add_comment(
  object = "don:core:dvrv-us-1:devo/xXjPo9nF:ticket/{id}",
  body = "Auto-assigned to {name} | Cohort: {cohort} | Pod: {pod}",
  visibility = "internal"
)
```

## Stage IDs

| Stage | ID | State | SLA Effect |
|-------|----|-------|------------|
| Queued | ordinal 500 | Open | Running |
| Reopen | custom_stage/90 | Open | Running |
| In Progress | custom_stage/4 | In Progress | Running |
| Awaiting Development | custom_stage/7 | In Progress | Running |
| Awaiting Customer Response | custom_stage/5 | In Progress | Paused |
| Resolved | custom_stage/1 | Closed | Stopped |
| Closed | custom_stage/81 | Closed | Stopped |

## Key Custom Fields

| Field | Type | Purpose |
|-------|------|---------|
| `tnt__customer_cohort_dropdown` | dropdown | Primary routing signal |
| `tnt__pod` | string | Sub-team routing |
| `tnt__assignee` | dev_user ref | Designated support person |
| `tnt__resolution` | string | How resolved |
| `tnt__resolved_by` | string | Resolved by Support/Engineering/Product |
| `tnt__work_duration` | string | Time spent (e.g., "1.07 days") |
| `tnt__ticket_type` | string | Support/Event/NCO/Retention |

## SLA

- Single policy: "Support Ticket SLA Default" (sla-28)
- Schedule: Mon-Fri 10AM-8PM IST (org_schedule-13)
- Metrics: First Response + Resolution Time
- Status: hit / miss

## Pod Values (from schema)
AI Agents, Alpha, Analytics, Brahmos, EXIM, Finance, Infra, Logistics Intelligence Team, Logistics Lighthouse, MCM, Mobile mad max, On Demand, POD, Platformization, Texas, WB, WMS, WMS Inbound, WMS Outbound
