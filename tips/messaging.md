# 💬 Messaging Channels — Setup & Patterns

OpenClaw supports Telegram, Discord, WhatsApp, Signal, Slack, iMessage, and more. This guide covers practical setup and multi-channel patterns.

## Telegram

### Setup (5 minutes)

1. Talk to [@BotFather](https://t.me/BotFather), run `/newbot`
2. Copy the token
3. Add to config:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123456:ABC-DEF",
      dmPolicy: "pairing",        // first-time users get a pairing code
      groups: {
        "*": { requireMention: true }  // only respond when @mentioned in groups
      }
    }
  }
}
```

4. Restart gateway, DM your bot, approve the pairing code

### Telegram Tips

- **BotFather settings matter:** Run `/setprivacy` to control whether the bot sees all group messages or only commands/mentions
- **Long-polling is default** — no webhook setup needed. It just works.
- **DMs share your main session** — your Telegram chat has the same context as webchat
- **Groups get isolated sessions** — each group gets `agent:main:telegram:group:<chatId>`
- **Topic support:** Telegram forum topics get separate sessions per topic

### Multi-account Telegram

Run multiple bots (e.g., personal + work):

```json5
{
  channels: {
    telegram: {
      accounts: [
        { name: "personal", botToken: "TOKEN_1" },
        { name: "work", botToken: "TOKEN_2" }
      ]
    }
  }
}
```

---

## Discord

### Setup

1. Create an app at [discord.com/developers](https://discord.com/developers/applications)
2. Bot → enable **Message Content Intent** and **Server Members Intent**
3. Copy the bot token
4. Generate an invite link with message read/send permissions
5. Config:

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

### Discord Tips

- **Mention gating:** By default, the bot only responds when @mentioned in guilds. Configure per-guild/channel in `channels.discord.guilds`
- **DMs use pairing** by default — unknown users get a code to approve
- **No markdown tables!** Discord renders them poorly. Use bullet lists instead
- **Suppress link embeds** with angle brackets: `<https://example.com>`
- **Native slash commands** auto-register by default — use `/config`, `/status`, etc.
- **History context:** Set `channels.discord.historyLimit` (default 20) to include recent messages as context when replying to mentions

### Guild-specific Rules

```json5
{
  channels: {
    discord: {
      guilds: {
        "123456789": {           // guild ID
          channels: {
            "general": { requireMention: false },  // respond to everything
            "bot-spam": { requireMention: false }
          }
        }
      }
    }
  }
}
```

---

## WhatsApp

### Setup

WhatsApp uses Baileys (WhatsApp Web protocol). You need a real phone number.

1. **Recommended:** Use a separate phone number (spare phone + eSIM is ideal)
2. Config:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]  // your personal number
    }
  }
}
```

3. Run `openclaw channels login` — scan the QR code with WhatsApp → Linked Devices
4. Start the gateway

### WhatsApp Tips

- **Use a dedicated number** — avoids conflicts with your personal WhatsApp
- **WhatsApp Business app** works great for the dedicated number on the same device
- **No markdown headers** — use **bold** or CAPS for emphasis
- **Allowlist is safer** than pairing for WhatsApp since anyone with your number can message
- **Media works** — the agent can receive and send images, documents, voice messages

---

## Multi-Channel Patterns

### Pattern 1: Single User, Multiple Channels

The most common setup. All DMs share your main session:

```json5
{
  channels: {
    telegram: { botToken: "...", dmPolicy: "pairing" },
    discord: { token: "...", dmPolicy: "pairing" },
    whatsapp: { dmPolicy: "allowlist", allowFrom: ["+1555..."] }
  }
}
```

**What this means:** You can start a conversation on Telegram, then continue it on Discord. The agent remembers everything because DMs share `agent:main:main`.

### Pattern 2: Multi-User Setup (Secure DM Isolation)

If multiple people DM your bot, **isolate their sessions:**

```json5
{
  session: {
    dmScope: "per-channel-peer"  // each user gets their own session
  }
}
```

⚠️ **Without this, all DMs share context.** Alice's private conversation leaks to Bob.

### Pattern 3: Identity Linking Across Channels

Same person on Telegram + Discord? Link their identities:

```json5
{
  session: {
    dmScope: "per-peer",
    identityLinks: {
      "telegram:12345": "alice",
      "discord:67890": "alice"
    }
  }
}
```

Now Alice gets the same session regardless of which channel she uses.

### Pattern 4: Group Chats Done Right

For groups, the agent should only speak when relevant:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }  // default: mention required
      }
    },
    discord: {
      guilds: {
        "*": { requireMention: true }
      }
    }
  }
}
```

**Pro tip:** In your AGENTS.md or system prompt, instruct the agent to stay quiet (`HEARTBEAT_OK`) when it has nothing meaningful to add. Quality > quantity.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Bot doesn't respond | Check `openclaw gateway status`, look for channel startup errors |
| "Pairing required" | Run `openclaw pairing approve <channel> <code>` |
| WhatsApp QR expired | Run `openclaw channels login` again |
| Discord missing messages | Enable **Message Content Intent** in dev portal |
| Messages going to wrong session | Check `session.dmScope` setting |
