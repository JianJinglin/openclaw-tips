# 🔒 Security & Sandboxing

Running an AI agent with shell access requires thoughtful guardrails. OpenClaw provides sandboxing, exec approvals, and tool policies to keep things safe.

## The Security Stack

```
Tool Policy          → What tools the agent can use
  ↓
Exec Approvals       → Which commands need human approval
  ↓
Sandboxing           → Where commands run (host vs container)
  ↓
Elevated Mode        → Escape hatch for trusted operations
```

## Sandboxing

Sandboxing runs tools inside Docker containers, limiting filesystem and process access.

### Modes

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",    // "off" | "non-main" | "all"
        scope: "session",    // "session" | "agent" | "shared"
        workspaceAccess: "none"  // "none" | "ro" | "rw"
      }
    }
  }
}
```

| Mode | Effect |
|------|--------|
| `off` | No sandboxing, tools run on host |
| `non-main` | Sandbox non-main sessions (groups, channels), main session runs on host |
| `all` | Everything sandboxed |

**Practical recommendation:** Use `non-main` if you trust your main chat but want group/Discord/Telegram sessions isolated. Use `all` for maximum paranoia.

### Scope

- **`session`** — one container per session (most isolated, uses more resources)
- **`agent`** — one container per agent
- **`shared`** — one container for everything (least isolated, lightest)

### Workspace Access

- **`none`** — sandbox gets its own workspace, can't see host files
- **`ro`** — host workspace mounted read-only at `/agent`
- **`rw`** — host workspace mounted read/write at `/workspace`

### Custom Bind Mounts

Mount specific host directories:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/user/projects:/projects:ro",
            "/var/run/docker.sock:/var/run/docker.sock"
          ]
        }
      }
    }
  }
}
```

## Exec Approvals

The companion app guardrail for commands running on real hosts. Three knobs:

### Security Policy

- **`deny`** — block all host exec
- **`allowlist`** — only allowlisted commands
- **`full`** — allow everything (use sparingly)

### Ask Mode

- **`off`** — never prompt
- **`on-miss`** — prompt when command isn't in allowlist (recommended)
- **`always`** — prompt on every command

### Ask Fallback

What happens when no UI is available to show the prompt:

- **`deny`** — block (safest)
- **`allowlist`** — allow if in allowlist
- **`full`** — allow everything

### Example Config

`~/.openclaw/exec-approvals.json`:

```json
{
  "version": 1,
  "defaults": {
    "security": "allowlist",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/*" },
        { "pattern": "~/Projects/**/node_modules/.bin/*" }
      ]
    }
  }
}
```

### Safe Bins

Some stdin-only tools are auto-allowed in allowlist mode: `jq`, `grep`, `cut`, `sort`, `uniq`, `head`, `tail`, `tr`, `wc`. They can only read from stdin, not access files directly.

### Approving from Chat

Forward approval prompts to Telegram/Discord/Slack and approve with `/approve`:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session",
      targets: [
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

Then reply: `/approve <id> allow-once` or `/approve <id> allow-always`

## Tool Policy

Control which tools the agent can access:

```json5
{
  tools: {
    exec: {
      security: "allowlist",   // deny | allowlist | full
      ask: "on-miss"
    },
    elevated: {
      enabled: false           // elevated runs on host, bypasses sandbox
    }
  }
}
```

## Practical Security Recipes

### Recipe 1: Personal Use (Trust Yourself)

```json5
{
  agents: {
    defaults: {
      sandbox: { mode: "non-main" }  // sandbox group chats only
    }
  }
}
```

Main session runs on host (you trust your own commands), group chats are sandboxed.

### Recipe 2: Multi-User Bot (Don't Trust Anyone)

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  },
  session: {
    dmScope: "per-channel-peer"
  }
}
```

Every session sandboxed, isolated, can't see host files. DMs isolated per user.

### Recipe 3: Development Server

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        workspaceAccess: "rw",
        docker: {
          binds: ["/home/dev/repos:/repos:rw"]
        }
      }
    }
  }
}
```

Sandboxed but with access to repos for coding tasks.

## Security Audit

Run `openclaw security audit` to check your configuration for common issues:

- Insecure DM policies
- Missing session isolation
- Overly permissive exec settings

## Key Principles

1. **Sandbox non-main by default** — group chats and external channels should be isolated
2. **Use allowlists over full access** — be explicit about what's allowed
3. **Isolate multi-user DMs** — set `dmScope: "per-channel-peer"` if more than one person can DM
4. **Approve from chat** — forward exec approvals to your phone for mobile approval
5. **`trash` > `rm`** — always prefer recoverable deletions
6. **Ask before sending** — the agent should confirm before sending emails, tweets, or public posts
