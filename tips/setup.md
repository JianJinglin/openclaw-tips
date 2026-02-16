# Setup & First Run

## Installation Options

### Fastest: Installer Script

```bash
# macOS / Linux / WSL2
curl -fsSL https://openclaw.ai/install.sh | bash

# Windows (PowerShell)
iwr -useb https://openclaw.ai/install.ps1 | iex
```

The script handles Node.js detection, installation, and launches the onboarding wizard.

### Manual: npm

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

> **Tip:** If you get `sharp` build errors on macOS, run:
> ```bash
> SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
> ```

### From Source (for contributors)

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install && pnpm ui:build && pnpm build
pnpm link --global
openclaw onboard --install-daemon
```

## System Requirements

- **Node.js 22+** (the installer handles this)
- macOS, Linux, or Windows (WSL2 recommended)
- On Windows: enable systemd in WSL2 (`/etc/wsl.conf` → `[boot] systemd=true`)

## First Run Checklist

### 1. Model Authentication

You need at least one LLM provider. During onboarding, pick your preferred provider:

```bash
# Or configure manually later
openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-20250514"
```

Set your API key via environment or config:
```bash
# Option A: .env file (recommended)
echo "ANTHROPIC_API_KEY=sk-ant-..." >> ~/.openclaw/.env

# Option B: config env block
openclaw config set env.ANTHROPIC_API_KEY "sk-ant-..."
```

### 2. Install the Gateway Daemon

```bash
openclaw gateway install
# or
openclaw onboard --install-daemon
```

This creates a systemd user service (Linux/WSL2) or LaunchAgent (macOS) that keeps the gateway running.

### 3. Verify Everything Works

```bash
openclaw status          # Overall health check
openclaw doctor          # Diagnose and fix issues
openclaw gateway status  # Gateway daemon status
```

### 4. Connect a Channel

See [Messaging](messaging.md) for channel-specific setup. Quick Telegram example:

```bash
openclaw configure  # Select Telegram, paste your bot token
```

## Where Things Live

| Path | Purpose |
|------|---------|
| `~/.openclaw/openclaw.json` | Main config file |
| `~/.openclaw/.env` | API keys and secrets |
| `~/.openclaw/workspace/` | Your agent's workspace (SOUL.md, memory, etc.) |
| `~/.openclaw/skills/` | Managed/local skills |
| `~/.openclaw/extensions/` | Plugins |

## Environment Variable Precedence

OpenClaw loads env vars from multiple sources (highest → lowest priority):

1. **Process environment** (parent shell)
2. **`.env` in current directory**
3. **`~/.openclaw/.env`** (global)
4. **Config `env` block** in `openclaw.json`
5. **Shell env import** (optional, for login shell vars)

**Key rule:** existing values are never overridden.

## Platform-Specific Notes

### Raspberry Pi

Perfect for a $35 always-on assistant. Use 64-bit OS, add 2GB swap, and stick with API-based models (don't try local LLMs on a Pi). See the [full Pi guide](https://docs.openclaw.ai/platforms/raspberry-pi).

### Oracle Cloud (Free Tier)

Up to 4 ARM CPUs + 24GB RAM for $0/month. Lock down the VCN to Tailscale-only traffic. See the [Oracle guide](https://docs.openclaw.ai/platforms/oracle).

### VPS General Pattern

```bash
# Install, bind to loopback, use Tailscale for remote access
openclaw config set gateway.bind loopback
openclaw config set gateway.auth.mode token
openclaw doctor --generate-gateway-token
openclaw config set gateway.tailscale.mode serve
```

## Helpful Commands

```bash
openclaw status --all     # Full status with provider usage
openclaw logs --follow    # Stream gateway logs
openclaw config show      # Dump current config
openclaw tui              # Terminal UI for direct chat
openclaw tui --deliver    # TUI with message delivery to channels
```
