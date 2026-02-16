# 🐯 OpenClaw Tips, Tricks & Best Practices

> Real-world patterns and lessons learned from running an autonomous AI assistant with [OpenClaw](https://openclaw.ai).

## What is OpenClaw?

OpenClaw is an open-source platform for building **autonomous AI assistants** that connect to your messaging channels (Telegram, Discord, WhatsApp, Slack, etc.), run on your own hardware, and have persistent memory, tool access, and proactive behavior via heartbeats and cron jobs.

Think of it as the infrastructure layer between an LLM (Claude, GPT, etc.) and your life — handling messaging, memory, browser automation, file access, scheduled tasks, sub-agents, and more.

## Why These Tips Matter

OpenClaw's docs are comprehensive but spread across many files. This repo distills **practical patterns** from actually running an autonomous assistant 24/7 — the kind of knowledge you only get from experience:

- How to structure workspace files so your AI remembers things across sessions
- Heartbeat and task queue patterns for truly autonomous work
- Sub-agent strategies that save tokens and parallelize research
- Security practices that keep your data safe
- Real config snippets you can copy-paste

## Table of Contents

| Guide | What You'll Learn |
|-------|-------------------|
| [Setup & First Run](tips/setup.md) | Installation, configuration, getting your first conversation going |
| [Autonomous Work](tips/autonomous-work.md) | HEARTBEAT.md, TASKQUEUE.md, cron jobs, sub-agents for 24/7 operation |
| [Memory System](tips/memory-system.md) | MEMORY.md, daily notes, long-term memory patterns |
| [Messaging](tips/messaging.md) | Telegram, Discord, WhatsApp, multi-channel setup |
| [Skills](tips/skills.md) | Using built-in skills, creating custom ones, ClawHub |
| [Browser Automation](tips/browser-automation.md) | Driving browsers, snapshots, Chrome extension relay |
| [Security](tips/security.md) | Sandboxing, exec approvals, access control, safety |
| [Advanced Patterns](tips/advanced.md) | Sub-agents, sessions, gateway config, plugins, Lobster workflows |
| [Examples](examples/) | Ready-to-use config snippets and workspace templates |

## Quick Start

```bash
# Install OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash

# Run onboarding (sets up model auth, channels, daemon)
openclaw onboard --install-daemon

# Open the TUI for direct chat
openclaw tui
```

Then read [Setup & First Run](tips/setup.md) for the full walkthrough.

## Contributing

Found a useful pattern? Open a PR! The goal is practical, battle-tested advice — not docs duplication.

## License

MIT
