# Memory System

Your AI wakes up fresh every session. Files are its memory. Here's how to make that work well.

## Architecture

```
~/.openclaw/workspace/
├── SOUL.md          # Who the AI is (personality, values, boundaries)
├── IDENTITY.md      # Name, creature type, emoji, avatar
├── USER.md          # About the human (preferences, projects, context)
├── MEMORY.md        # Long-term curated memories
├── AGENTS.md        # Session startup instructions
├── TOOLS.md         # Environment-specific notes (cameras, SSH hosts, etc.)
├── HEARTBEAT.md     # Heartbeat check-in logic
├── TASKQUEUE.md     # Work queue
└── memory/
    ├── 2026-02-15.md    # Daily raw notes
    ├── 2026-02-14.md
    └── heartbeat-state.json
```

## Session Boot Sequence

Every session, the agent should (per AGENTS.md):

1. Read `SOUL.md` — who it is
2. Read `USER.md` — who it's helping
3. Read `memory/YYYY-MM-DD.md` (today + yesterday) — recent context
4. **Main session only:** Read `MEMORY.md` — long-term memory

> ⚠️ **Security:** MEMORY.md contains personal context. Only load it in main sessions (direct chat with your human). Never load it in group chats or shared contexts.

## Daily Notes (`memory/YYYY-MM-DD.md`)

Raw logs of what happened. Write throughout the day:

```markdown
# 2026-02-15

## Morning
- Ollie asked about diffusion model papers
- Found 3 relevant papers, sent summaries
- Started draft of research proposal section 3

## Afternoon
- Completed proposal draft, sent for review
- Updated TASKQUEUE.md
- Background: ran paper survey sub-agent

## Decisions
- Chose Flow Matching over DDPM for the proposal (better for continuous data)
- Paused Ge Liu collaboration per strategy discussion
```

**Tips:**
- Create `memory/` directory if it doesn't exist
- Keep entries concise but useful for future-you
- Include decisions and their reasoning
- Note what was sent to the user vs. done silently

## Long-Term Memory (`MEMORY.md`)

Curated, distilled insights. Not a log — a living document:

```markdown
# MEMORY.md

## Key Preferences
- Ollie prefers direct answers, no filler
- Bilingual: match their language (EN/中文)
- Morning briefs via Telegram at 8am PST

## Project Context
- PhD at Scripps, advisor Stefano Forli
- Three research threads: own lab, Stanford intern, Rose Yu collab
- Paper output is THE core metric

## Lessons Learned
- Sub-agents work best with specific, bounded tasks
- Always update TASKQUEUE.md after completing work
- Don't suggest more than 3 action items at once
```

### Memory Maintenance (Weekly)

During a heartbeat, periodically:

1. Read through recent `memory/YYYY-MM-DD.md` files
2. Identify significant events, lessons, insights
3. Update `MEMORY.md` with distilled learnings
4. Remove outdated info from MEMORY.md

**Think:** journal review → mental model update.

## SOUL.md — Personality & Values

This is who your AI is. A real-world example:

```markdown
# SOUL.md

## Core Truths
- Be genuinely helpful, not performatively helpful
- Be proactive — anticipate needs
- Have opinions
- Be resourceful before asking
- Earn trust through competence

## Boundaries
- Private things stay private
- Ask before acting externally
- Never send half-baked replies
- You're not the user's voice in group chats

## Vibe
Efficient AND warm. Get things done fast but don't be a cold robot.
```

**Key insight:** Let the AI evolve this file. It should update SOUL.md as it learns who it is — but always tell the user when it changes.

## USER.md — Know Your Human

```markdown
# USER.md

- Name: Ollie
- Pronouns: she/her
- Timezone: PST
- Notes: Bilingual EN/中文, INTJ/ENTJ, values efficiency

## Current Projects
- PhD research (AI × Science)
- Paper deadlines: ICLR Sep 24

## Daily Preferences
- Morning brief via Telegram
- Sleep ~1-2am, wake ~8-9am PST
```

**Pattern:** Update USER.md continuously as you learn about the person. Projects, preferences, communication style — all here.

## TOOLS.md — Environment Notes

Skills define *how* tools work. TOOLS.md is for *your* specifics:

```markdown
# TOOLS.md

### GitHub
- Username: myuser
- Token: ghp_... (repo scope)

### Cameras
- living-room → Main area, 180° wide angle

### TTS
- Preferred voice: "Nova"
```

## The "Write It Down" Rule

> **Memory is limited. If you want to remember something, WRITE IT TO A FILE.**

- "Mental notes" don't survive session restarts. Files do.
- When someone says "remember this" → update daily notes or relevant file
- When you learn a lesson → update AGENTS.md or TOOLS.md
- When you make a mistake → document it so future-you doesn't repeat it

## Memory Search (Plugin)

OpenClaw has built-in memory plugins:

- **memory-core** (default): basic memory search
- **memory-lancedb**: long-term vector memory with auto-recall/capture

Configure in `openclaw.json`:
```json
{
  "plugins": {
    "slots": {
      "memory": "memory-core"
    }
  }
}
```
