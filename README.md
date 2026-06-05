# Production Claude Agents

A collection of scheduled Claude agent skills running in production. Each agent is a SKILL.md file — a structured prompt that Claude Code executes autonomously on a cron schedule using the built-in `schedule` skill.

These are real agents doing real work daily. The prompts are shared as reference for anyone building similar agentic workflows with Claude Code.

## Agents

### [`daily-briefing/`](daily-briefing/SKILL.md) — Daily Executive Briefing

Fires every weekday morning. Pulls Calendar, Slack, Gmail, the project board, and the issue tracker, applies a strict relevance filter (only items directly addressed to the user, @mentioned, or explicitly assigned), and writes a prioritized markdown briefing to the working folder before the workday starts.

**Output structure:**
- Today at a Glance — meetings table + heads-down windows of 30+ min
- Prioritized Action Plan — Urgent / High / Medium / Low tiers, each item with source link, time estimate, and numbered next steps

**Why it works:** The relevance filter is the critical piece. Without it, the briefing becomes a noise dump. The filter excludes bulk distribution lists, OOO auto-replies, automated system notifications, and recurring digests by rule — not by judgment call on each run.

---

### [`email-triage/`](email-triage/SKILL.md) — Intra-day Email Triage

Runs multiple times daily. Labels the inbox with Action Needed, Follow-Up, and Resolved; compiles a structured action-item list cross-referenced against the project board; checks escalation triggers (threads aging past thresholds, unacknowledged stakeholder emails); and queues reply drafts.

**Hard rules enforced in the prompt:**
- Drafts only — never sends
- No deletion, no archiving during triage
- Fixed output format every run (inbox count → action items → stakeholder flags → drafts queued → resolved count)

---

### [`eod-summary/`](eod-summary/SKILL.md) — EOD Priorities Draft

Fires each weekday afternoon. Gathers the day's activity from Slack, Gmail, the project board, Confluence, and local Claude sessions — organizes it into a numbered priorities list — and saves a rendered HTML Gmail draft ready to review and send.

**Why draft-first:** Eliminates the writing step while preserving editorial control. The agent does the aggregation and organization; you review and send.

---

### [`weekly-status-updates/`](weekly-status-updates/SKILL.md) — Weekly Project Status Drafts

Runs weekly. Pulls every open project from the board, gathers context from Gmail, Drive, and Slack in parallel for each project, and generates a branded HTML status email per project saved as a Gmail draft addressed to the correct stakeholders.

**Source priority:** Gmail (most recent thread) → Drive (modified docs) → Monday (milestone dates). Overdue items with no recent updates are flagged before drafts are generated.

---

## How these work

Each agent is a SKILL.md file with a `name` and `description` in the frontmatter, followed by the full prompt. Claude Code's `schedule` skill reads these files and executes them on the configured cron.

```
/schedule  →  picks up SKILL.md  →  executes on cron  →  writes output / creates drafts / posts to Slack
```

To adapt these for your own setup:
1. Replace the placeholder values (board IDs, channel IDs, email addresses, timezone)
2. Adjust the connected tools/MCPs to match your stack
3. Schedule via `/schedule` in Claude Code

## Patterns worth noting

**Relevance filters before reads.** The daily briefing defines what to exclude before touching any API. This prevents the agent from spending tokens on noise it will discard anyway.

**Draft-first for any outbound communication.** The EOD summary and weekly status agents never send — they create drafts. This keeps a human in the loop on anything client- or leadership-facing without adding writing overhead.

**Fixed output formats.** The email triage agent produces the same structure every run. Consistent format means the output is scannable in seconds rather than requiring re-orientation each time.

**Parallel context gathering.** The weekly status agent queries Gmail, Drive, and Slack simultaneously for each project rather than sequentially. On a portfolio of several projects, this is the difference between a 2-minute run and a 10-minute run.

**Source priority hierarchy.** When multiple sources disagree on project status, the agents follow Gmail (most recent human-written thread) > Drive (current doc state) > project board (milestone dates only). Explicit hierarchy prevents the agent from surfacing stale board data over a recent email thread.

## License

MIT
