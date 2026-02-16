# 🔧 Config Snippets

Useful `openclaw.json` snippets for common setups. Config lives at `~/.openclaw/openclaw.json`.

## Minimal Single-User Setup

```json5
{
  models: {
    default: "anthropic/claude-sonnet-4-20250514"
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_TOKEN",
      dmPolicy: "pairing"
    }
  }
}
```

## Full Personal Assistant

```json5
{
  models: {
    default: "anthropic/claude-sonnet-4-20250514"
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_TELEGRAM_TOKEN",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    },
    discord: {
      enabled: true,
      token: "YOUR_DISCORD_TOKEN"
    },
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  },
  heartbeat: {
    enabled: true,
    intervalMinutes: 30
  },
  browser: {
    enabled: true,
    defaultProfile: "openclaw"
  },
  agents: {
    defaults: {
      sandbox: { mode: "non-main" },
      subagents: {
        model: "anthropic/claude-sonnet-4-20250514",
        archiveAfterMinutes: 60
      }
    }
  }
}
```

## Multi-User Bot (Secure)

```json5
{
  session: {
    dmScope: "per-channel-peer"
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_TOKEN",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } }
    },
    discord: {
      enabled: true,
      token: "YOUR_TOKEN",
      dm: {
        policy: "pairing"
      }
    }
  },
  agents: {
    defaults: {
      sandbox: {
        mode: "all",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

## Development / Coding Agent

```json5
{
  models: {
    default: "anthropic/claude-sonnet-4-20250514"
  },
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
        sandbox: {
          mode: "all",
          workspaceAccess: "rw",
          docker: {
            binds: ["~/Projects:/projects:rw"]
          }
        }
      }
    ]
  },
  browser: {
    enabled: true,
    headless: true,
    defaultProfile: "openclaw"
  }
}
```

## Headless VPS / Server

```json5
{
  gateway: {
    port: 18789,
    host: "127.0.0.1"    // loopback only; use Tailscale for remote
  },
  models: {
    default: "anthropic/claude-sonnet-4-20250514"
  },
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_TOKEN",
      dmPolicy: "allowlist",
      allowFrom: ["YOUR_TELEGRAM_ID"]
    }
  },
  browser: {
    enabled: true,
    headless: true,
    noSandbox: true,
    defaultProfile: "openclaw"
  },
  agents: {
    defaults: {
      sandbox: { mode: "all" }
    }
  }
}
```

## Cost-Optimized Setup

```json5
{
  models: {
    default: "anthropic/claude-haiku-3-20250618",   // cheap default
    failover: ["anthropic/claude-sonnet-4-20250514"]  // fallback
  },
  agents: {
    defaults: {
      subagents: {
        model: "anthropic/claude-haiku-3-20250618"  // cheap sub-agents
      }
    }
  },
  heartbeat: {
    enabled: true,
    intervalMinutes: 60    // less frequent = fewer tokens
  }
}
```

## Exec Approvals (Chat-Based)

Forward approval prompts to your phone:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "both",
      targets: [
        { channel: "telegram", to: "YOUR_CHAT_ID" }
      ]
    }
  }
}
```

## Custom Browser Profiles

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "openclaw",
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work:     { cdpPort: 18801, color: "#0066CC" },
      remote:   { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" }
    }
  }
}
```
