# 🌐 Browser Automation

OpenClaw gives the agent a dedicated browser it can control — open pages, click buttons, fill forms, take screenshots. Two modes: managed (`openclaw` profile) or Chrome extension relay (`chrome` profile).

## Two Profiles

| Profile | What it is | When to use |
|---------|-----------|-------------|
| `openclaw` | Isolated browser managed by OpenClaw | Automation, scraping, testing |
| `chrome` | Your real Chrome via extension relay | Accessing logged-in sites, your cookies |

Set the default:

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "openclaw"  // or "chrome"
  }
}
```

## Quick Start (Managed Browser)

```bash
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

The managed browser launches a separate Chromium profile (orange accent by default) that won't touch your personal browser data.

## Chrome Extension Relay

For accessing sites where you're already logged in:

1. Install the OpenClaw Browser Relay extension
2. Navigate to the target page in your browser
3. Click the extension icon (badge turns ON)
4. Now the agent can see and interact with that tab

**Key insight:** The `chrome` profile doesn't control your whole browser — only the tab you explicitly attach via the extension.

## How the Agent Uses the Browser

### Snapshots (the main tool)

Snapshots return an accessibility tree of the page — a structured representation of every interactive element. The agent reads this to understand the page.

```
# Agent sees something like:
[ref=e1] button "Sign In"
[ref=e2] textbox "Email" value=""
[ref=e3] textbox "Password" value=""
```

The agent then acts on refs:

```
browser action: click ref=e1
browser action: fill ref=e2 text="user@example.com"
```

### Screenshots

For visual verification or when the accessibility tree isn't enough:

```
browser action: screenshot
```

### Common Workflow

1. **Navigate** to URL
2. **Snapshot** to understand the page
3. **Act** (click, type, fill)
4. **Snapshot** again to verify result
5. Repeat

## Configuration

```json5
{
  browser: {
    enabled: true,
    defaultProfile: "openclaw",
    headless: false,              // true for servers
    noSandbox: false,             // true if running as root (Linux)
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work:     { cdpPort: 18801, color: "#0066CC" },
      remote:   { cdpUrl: "http://10.0.0.42:9222" }  // browser on another machine
    }
  }
}
```

### Multiple Profiles

Use multiple profiles for different contexts — personal, work, testing:

```json5
{
  browser: {
    profiles: {
      openclaw: { cdpPort: 18800 },
      work: { cdpPort: 18801, color: "#0066CC" },
    }
  }
}
```

## Practical Patterns

### Pattern 1: Research with web_fetch First

Don't reach for the browser immediately. `web_fetch` is faster and cheaper for reading content:

```
# Fast path — no browser needed
web_fetch url="https://docs.example.com/api" extractMode="markdown"

# Only use browser when you need interaction or JavaScript-rendered content
```

### Pattern 2: Login Flows

For sites requiring login, use the Chrome extension relay:

1. Log in manually in your browser
2. Attach the tab via extension
3. Agent uses `profile="chrome"` to access the authenticated session

### Pattern 3: Sandboxed Browser

When running in sandbox mode, the browser can also run inside the container:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        browser: {
          autoStart: true,
          allowHostControl: false  // keep it isolated
        }
      }
    }
  }
}
```

### Pattern 4: Headless on Servers

For VPS/CI environments:

```json5
{
  browser: {
    headless: true,
    noSandbox: true,  // often needed in containers
    defaultProfile: "openclaw"
  }
}
```

## Tips & Gotchas

- **Snapshot > Screenshot** — snapshots give structured data the model can reason about; screenshots are just pixels
- **Keep the same tab** — when acting on refs from a snapshot, pass `targetId` to stay on the same tab
- **`refs="aria"`** gives stable IDs across snapshots; `refs="role"` (default) uses role+name which can change
- **Don't overuse `act:wait`** — only use when no reliable UI state exists to check
- **Browser auto-detects** Chromium-based browsers: Chrome → Brave → Edge → Chromium
- **CDP ports** are auto-derived from gateway port (default: 18791 for control, 18792 for relay)
