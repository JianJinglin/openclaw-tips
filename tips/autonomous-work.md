# Autonomous Work Patterns

The real power of OpenClaw is **autonomous operation** — your AI assistant working proactively, not just responding to messages.

## The Three Pillars

### 1. HEARTBEAT.md — Periodic Check-Ins

The heartbeat system polls your agent at regular intervals. When triggered, the agent reads `HEARTBEAT.md` and decides whether to take action or reply `HEARTBEAT_OK`.

**Real-world HEARTBEAT.md example:**

```markdown
# HEARTBEAT.md — Heartbeat Logic

## Instructions
1. Read TASKQUEUE.md
2. If NOW has a task → continue working on it
3. If NOW is empty → move top NEXT item to NOW → start it
4. If NEXT is empty → check BACKLOG for something useful
5. If nothing to do → HEARTBEAT_OK

## Constraints
- Night mode (11pm-8am PST): NO messages, only silent work
- Don't send more than 3 unsolicited messages per hour (daytime)
- Use sub-agents for heavy research (don't block main session)
- Always update TASKQUEUE.md after completing a task

## Self-Check (every 5th heartbeat)
- Am I making progress or spinning?
- Is my current task still the highest priority?
```

**Key insight:** Keep HEARTBEAT.md small. It's loaded every heartbeat, so every token counts.

### 2. TASKQUEUE.md — Work Queue

Structure your agent's work as a prioritized queue:

```markdown
# TASKQUEUE.md — Work Queue

## 🔴 NOW
- Writing research proposal draft (section 3)

## 🟡 NEXT
- Review latest papers on diffusion models
- Update project dashboard

## 🟢 BACKLOG
- Draft blog post series
- Security hardening review
- Content strategy discussion

## ✅ DONE (recent)
- Deep analysis of paper X ✅
- 7-day learning plan ✅
- Research proposal sections 1-2 ✅

## 📋 RECURRING
- [ ] Morning brief (8:00 AM) — cron handles
- [ ] Memory maintenance — 1x/week
```

**Pattern:** The agent moves items through the pipeline: BACKLOG → NEXT → NOW → DONE.

### 3. Sub-Agents — Parallel Workers

For heavy tasks, spawn sub-agents instead of blocking the main session:

```
# The agent uses sessions_spawn internally
Task: "Research the top 10 papers on protein diffusion models from 2025"
Label: "protein-diffusion-survey"
Model: "anthropic/claude-sonnet-4-20250514"  # Use cheaper model for research
```

**Sub-agent tips:**
- Sub-agents get their own context and token budget
- They announce results back to the requester chat when done
- Use cheaper models for sub-agents (`agents.defaults.subagents.model`)
- Max concurrent: `agents.defaults.subagents.maxConcurrent` (default 8)
- Sub-agents **cannot** spawn sub-agents (no nested fan-out)
- They only get AGENTS.md + TOOLS.md (no SOUL.md or USER.md)

### Heartbeat vs Cron: When to Use Which

| Use Case | Heartbeat | Cron |
|----------|-----------|------|
| Batch multiple checks together | ✅ | |
| Needs conversational context | ✅ | |
| Exact timing matters | | ✅ |
| Isolated from main session | | ✅ |
| Different model/thinking level | | ✅ |
| One-shot reminders | | ✅ |

**Pro tip:** Batch similar periodic checks into HEARTBEAT.md instead of creating multiple cron jobs. Use cron for precise schedules and standalone tasks.

## Proactive Work (What to Do Without Asking)

Safe to do autonomously:
- Read and organize memory files
- Check on projects (git status, etc.)
- Update documentation
- Commit and push changes
- Review and update MEMORY.md

Things to check (rotate through 2-4x/day):
- Emails — urgent unread?
- Calendar — upcoming events in 24-48h?
- Mentions — social notifications?
- Weather — relevant for plans?

## Night Mode Pattern

```markdown
## Night Mode (11pm-8am)
- NO messages to user unless urgent
- Silent work only: organizing, writing, research
- Queue findings for morning brief
- Update TASKQUEUE.md progress
```

## Tracking Heartbeat State

Use a JSON file to avoid redundant checks:

```json
{
  "lastChecks": {
    "email": 1703275200,
    "calendar": 1703260800,
    "weather": null
  }
}
```

Save to `memory/heartbeat-state.json` and update timestamps after each check.

## Anti-Patterns

❌ **Spinning:** Checking the same thing every heartbeat with no new info
❌ **Blocking:** Running a 10-minute research task in the main session
❌ **Over-messaging:** Sending 5 updates when 1 summary would do
❌ **Forgetting to update:** Finishing a task but not moving it to DONE
❌ **Giant HEARTBEAT.md:** Adding too much logic; keep it under 30 lines
