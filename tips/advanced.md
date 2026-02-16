# ⚡ Advanced — Sub-Agents, Sessions, Gateway & More

## Sub-Agents

Sub-agents are background agent runs that work in parallel, then report back. Think of them as async tasks.

### When to Use Sub-Agents

- **Research tasks** — "look up X, Y, Z simultaneously"
- **Long-running work** — "build this while I keep chatting"
- **Isolation** — sub-agents get their own session, can't mess up your main context

### How They Work

The agent calls `sessions_spawn` with a task description. The sub-agent runs in `agent:<agentId>:subagent:<uuid>`, then announces results back to the original chat.

```
Main agent → spawns sub-agent with task
Sub-agent → works in isolation
Sub-agent → announces result back to chat
```

### Key Facts

- **No nesting** — sub-agents can't spawn sub-agents (prevents fan-out bombs)
- **Separate token budget** — each sub-agent has its own context
- **Auto-archive** — sessions auto-delete after 60 minutes by default
- **Cost control** — use a cheaper model for sub-agents:

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "anthropic/claude-sonnet-4-20250514",  // cheaper than main
        archiveAfterMinutes: 60
      }
    }
  }
}
```

### Managing Sub-Agents

```
/subagents list              # see running sub-agents
/subagents stop <id|all>     # kill one or all
/subagents log <id> [limit]  # view transcript
/subagents info <id>         # metadata, timing, cost
/subagents send <id> <msg>   # send message to running sub-agent
```

### Sub-Agent Auth

Sub-agents inherit the spawning agent's auth as a fallback, so they can access the same APIs and services. Agent-specific auth profiles override on conflicts.

---

## Sessions

### Session Keys

Every conversation maps to a session key:

| Source | Session Key |
|--------|------------|
| Main DM | `agent:main:main` |
| Telegram group | `agent:main:telegram:group:<chatId>` |
| Discord channel | `agent:main:discord:channel:<channelId>` |
| Sub-agent | `agent:main:subagent:<uuid>` |
| Slash command | `agent:main:discord:slash:<userId>` |

### DM Scoping (Critical for Multi-User)

```json5
{
  session: {
    dmScope: "main"                // all DMs share one session (default)
    // dmScope: "per-channel-peer"  // isolated per user per channel (secure)
    // dmScope: "per-peer"          // isolated per user across channels
  }
}
```

### Session Pruning

OpenClaw auto-trims old tool results from context before LLM calls. This doesn't rewrite history — it just drops verbose tool outputs from older turns to save tokens.

### Pre-Compaction Memory Flush

When a session approaches compaction, OpenClaw can run a silent memory flush — saving important context to memory files before the old messages get summarized.

---

## Gateway Configuration

The gateway is the core daemon. Key config areas:

### Essential Settings

```json5
// ~/.openclaw/openclaw.json
{
  gateway: {
    port: 18789,              // default
    // host: "0.0.0.0",       // bind to all interfaces (for remote access)
  },
  
  // Model configuration
  models: {
    default: "anthropic/claude-sonnet-4-20250514",
    providers: {
      anthropic: {
        apiKey: "sk-ant-..."
      },
      openai: {
        apiKey: "sk-..."
      }
    }
  }
}
```

### Model Failover

Configure fallback models when your primary is down:

```json5
{
  models: {
    default: "anthropic/claude-sonnet-4-20250514",
    failover: [
      "openai/gpt-4o",
      "anthropic/claude-haiku-3-20250618"
    ]
  }
}
```

### Multiple Gateways

Run multiple gateway instances for different agents or contexts:

```bash
OPENCLAW_GATEWAY_PORT=18789 openclaw gateway start  # primary
OPENCLAW_GATEWAY_PORT=18800 openclaw gateway start  # secondary
```

### Heartbeat Configuration

```json5
{
  heartbeat: {
    enabled: true,
    intervalMinutes: 30,
    prompt: "Read HEARTBEAT.md if it exists. Follow it strictly. If nothing needs attention, reply HEARTBEAT_OK."
  }
}
```

---

## Cron Jobs

Schedule one-off or recurring tasks:

```
/cron add "0 9 * * 1" "Check my email inbox and summarize anything important"
/cron add "*/30 * * * *" "Check RSS feeds for AI news"
/cron list
/cron remove <id>
```

### Cron vs Heartbeat

| Feature | Heartbeat | Cron |
|---------|-----------|------|
| Timing | Approximate (~30min) | Exact (cron expression) |
| Context | Shares main session history | Isolated session |
| Model | Same as main | Can specify different model |
| Best for | Batched periodic checks | Precise schedules, reminders |

**Tip:** Batch similar checks into HEARTBEAT.md instead of creating multiple cron jobs. Use cron for exact timing and standalone tasks.

---

## Multi-Agent Setup

Run multiple agents with different personalities or roles:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        workspace: "~/.openclaw/workspace"
      },
      {
        id: "coder",
        workspace: "~/Projects",
        model: "anthropic/claude-sonnet-4-20250514",
        sandbox: { mode: "all", workspaceAccess: "rw" }
      },
      {
        id: "researcher",
        model: "anthropic/claude-haiku-3-20250618",  // cheaper for high-volume
        sandbox: { mode: "all" }
      }
    ]
  }
}
```

### Cross-Agent Spawning

Allow one agent to spawn sub-agents under another agent's identity:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        subagents: {
          allowAgents: ["coder", "researcher"]  // main can spawn these
        }
      }
    ]
  }
}
```

---

## Skills (Plugins)

Skills extend the agent's capabilities with external tools:

```bash
openclaw skills install <skill-name>
openclaw skills list
```

Skills provide:
- Additional CLI tools the agent can use
- SKILL.md files with usage instructions
- Auto-allowed binaries (when `autoAllowSkills` is enabled in exec approvals)

### Skill Config

```json5
{
  skills: {
    enabled: true,
    autoInstall: true,  // install missing skills automatically
    list: ["sag", "calendar", "email"]
  }
}
```

---

## Useful CLI Commands

```bash
# Gateway management
openclaw gateway status
openclaw gateway start
openclaw gateway stop
openclaw gateway restart

# Channel management
openclaw channels login          # WhatsApp QR login
openclaw channels status         # see all channels

# Security
openclaw security audit          # check config for issues
openclaw pairing approve <ch> <code>  # approve DM pairing

# Sessions
openclaw sessions list
openclaw sessions delete <key>

# Browser
openclaw browser status
openclaw browser start
openclaw browser open <url>

# Debugging
openclaw doctor                  # diagnose common issues
openclaw logs                    # view gateway logs
```
