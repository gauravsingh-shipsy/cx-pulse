# CX Pulse — Support Intelligence & Auto-Allocation

Proactive daily alerting + intelligent ticket routing for Shipsy's customer support.

## What This Project Does

1. **Morning Brief (8 AM IST)** — pulls DevRev tickets, roster, Slack channel signals. Posts structured report to Slack.
2. **Evening Wrap-up (9 PM IST)** — resolved today, SLA performance, carry-over, tomorrow's roster.
3. **Sentinel (every 2h)** — checks war rooms, incidents, blocker tickets, frustrated customer patterns. Silent if all clear.
4. **Auto-Allocation (Phase 2)** — when a support ticket arrives in DevRev, routes it to the right person based on cohort, pod, capacity, and roster.

## Stack

- Runtime: Node.js >= 18
- AI: Claude Agent SDK (`@anthropic-ai/claude-agent-sdk`)
- MCPs: DevRev, Slack, Google Drive, New Relic, GitHub
- Deployment: AWS ECS (same cluster as Shipra)
- Triggers: Claude Code Remote Triggers (cron-scheduled)

## Key Directories

- `.claude/skills/` — CX Pulse skills (morning-brief, evening-wrap, sentinel, allocate-ticket)
- `.claude/agents/` — isolated agents (deep-research, pattern-analyzer)
- `.claude/rules/` — domain knowledge (cohort mapping, team pools, DevRev API reference)
- `config/` — runtime config (cohort-map.json, team-roster.json, thresholds.json)
- `docs/` — architecture plans, analysis documents
- `src/` — allocation engine source code (Phase 2)

## MCP Servers

Defined in `.mcp.json`. Required connections:
- **DevRev** — source of truth for tickets, SLA, sentiment, cohorts
- **Slack** — read channels (140+), post reports, send DMs
- **Google Drive** — roster sheet (`1v8lbH2yZCU7TAInUNO2tqx-HDGbCjPVtqZEWX94pApc`)

## Key DevRev Fields

- `tnt__customer_cohort_dropdown` — primary routing signal (20 values)
- `tnt__pod` — sub-team routing (Alpha, Brahmos, On Demand, MCM, WMS, Finance, etc.)
- `owned_by` — ticket owner
- `tnt__assignee` — designated support person
- `group` — DevRev group (Reliance Support, DTDC Support, On demand POD, etc.)
- `sentiment.label` — Happy/Neutral/Unhappy/Frustrated
- `sla_summary` — hit/miss for First Response and Resolution Time

## Report Target

Slack channel: `C0A82U7MZ5F` (leadership group DM)

## Conventions

- Skills use `.claude/skills/<name>/SKILL.md` with frontmatter
- Agents use `.claude/agents/<name>.md` with frontmatter
- Domain knowledge goes in `.claude/rules/`
- Config files are JSON in `config/`
- Never fabricate DevRev data — only report what APIs return
- Ticket IDs must be clickable DevRev links in Slack messages
