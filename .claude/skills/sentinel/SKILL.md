---
name: sentinel
description: Real-time proactive alerts — runs every 2 hours, checks war rooms, incidents, blocker tickets, frustrated patterns, New Relic spikes. Posts to C0A82U7MZ5F and C07BQD5776Y ONLY if something needs attention. Silent if all clear.
---

# Sentinel — CX Pulse

Run every 2 hours. Post ONLY if something needs attention. Silent if all clear.
Post to BOTH: C0A82U7MZ5F and C07BQD5776Y.

## What to Check (last 2 hours)

### 1. Incident Channels
- `C033JL0BVN0` (#war-room)
- `C09A4PG1RTP` (#war-room-dtdc)
- `C02NVJ91BT6` (#incidents)
- `C09SXD56MQS` (#incident-io-incidents)
- `C07KDUT9D5M` (#support-blocker-p1-tickets)
- `C09K1LGF4N9` (#aramex-stability-issues)

### 2. Frustrated Patterns
`C07BQD5776Y` — if >= 3 alerts from SAME account in 2 hours = pattern.

### 3. New Blocker/High Tickets
DevRev: `hybrid_search namespace=ticket query="blocker high severity created recently"`

### 4. New Relic (if available)
Error rate spikes, latency anomalies. Skip silently if not connected.

## Decision

```
IF war_room active         → alert
IF new incident            → alert
IF blocker ticket in 2h    → alert
IF >= 3 frustrated same acct → alert
IF error spike             → alert
IF nothing found           → EXIT SILENTLY
```

## Alert Format

```
*CX Pulse — Sentinel Alert* | {{time}} IST

*{{SIGNAL TYPE}}* {{color}}
> {{Details: ticket/incident, account, what happened}}
> {{Link to war room / ticket}}

*Support Impact:* {{one-liner}}
*Action:* {{who should do what}}
```

## Rules
1. SILENT if nothing found. No "all clear" messages.
2. Under 1500 characters.
3. War room active = always include channel link.
4. Deploy + errors = correlate them.
