# 💓 Example HEARTBEAT.md

Drop this in your workspace (`~/.openclaw/workspace/HEARTBEAT.md`) and customize.

```markdown
# HEARTBEAT.md

## Checks (rotate through these, don't do all every time)

### Priority 1 — Every heartbeat
- [ ] Check memory/YYYY-MM-DD.md exists for today; create if not

### Priority 2 — Every 2-3 hours
- [ ] Check email inbox for urgent/unread (skill: email)
- [ ] Check calendar for events in the next 4 hours

### Priority 3 — 2x daily
- [ ] Check weather if going-out plans exist in calendar
- [ ] Quick git status on active projects — commit uncommitted work
- [ ] Review and tidy workspace files

### Priority 4 — Weekly (Monday morning)
- [ ] Review memory/ files from past week
- [ ] Update MEMORY.md with key learnings
- [ ] Clean up old memory files (>30 days)
- [ ] Check for OpenClaw updates

## Rules
- Late night (23:00-08:00): HEARTBEAT_OK unless truly urgent
- If nothing new since last check: HEARTBEAT_OK
- Track check timestamps in memory/heartbeat-state.json
- Don't repeat work you did <30 minutes ago

## Active Reminders
<!-- Add temporary reminders here, remove when done -->
- Remind David about dentist appointment on Thursday
- Follow up on PR #42 if not merged by Wednesday
```

## How It Works

The heartbeat system polls your agent periodically (default: every 30 minutes). The agent reads this file and decides what to do. If nothing needs attention, it replies `HEARTBEAT_OK` and the system stays quiet.

## Tips

- **Keep it small** — every heartbeat burns tokens reading this file
- **Use priorities** — not everything needs checking every 30 minutes
- **Track state** — use `memory/heartbeat-state.json` to avoid redundant checks:

```json
{
  "lastChecks": {
    "email": "2025-02-15T10:00:00Z",
    "calendar": "2025-02-15T10:00:00Z",
    "weather": "2025-02-15T06:00:00Z",
    "git": "2025-02-14T22:00:00Z"
  }
}
```

- **Temporary reminders** — add and remove from the "Active Reminders" section as needed
- **Don't over-engineer** — start simple, add checks as you discover what's useful
