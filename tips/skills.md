# Skills

Skills teach your AI how to use specific tools and domains. They're modular, hot-reloadable, and shareable.

## How Skills Work

A skill is a directory with a `SKILL.md` file containing YAML frontmatter (metadata) and markdown instructions. OpenClaw injects eligible skills into the system prompt.

```
my-skill/
├── SKILL.md           # Required: metadata + instructions
├── scripts/           # Optional: executable code
├── references/        # Optional: docs loaded on demand
└── assets/            # Optional: templates, images
```

## Skill Locations (Precedence)

1. **`<workspace>/skills/`** — highest priority, per-agent
2. **`~/.openclaw/skills/`** — shared across agents
3. **Bundled skills** — shipped with OpenClaw
4. **`skills.load.extraDirs`** — extra folders (lowest)

Same-name skills: workspace wins → managed → bundled.

## Built-in Skills (Examples)

OpenClaw ships with many skills. Some highlights:

| Skill | What It Does |
|-------|-------------|
| `github` | GitHub operations via CLI |
| `notion` | Notion API integration |
| `himalaya` | Email via himalaya CLI |
| `peekaboo` | macOS camera snapshots |
| `spotify-player` | Spotify control |
| `coding-agent` | Run Codex/Claude Code/Pi via background process |
| `canvas` | Render HTML/CSS/JS on device canvases |
| `nano-pdf` | PDF generation |
| `skill-creator` | Create new skills (meta!) |

## Creating a Custom Skill

### 1. Create the Directory

```bash
mkdir -p ~/.openclaw/workspace/skills/my-tool
```

### 2. Write SKILL.md

```markdown
---
name: my_tool
description: Brief description of what this skill does and when to use it.
metadata: {"openclaw": {"requires": {"bins": ["mytool"]}, "primaryEnv": "MY_API_KEY"}}
---

# My Tool

Instructions for the AI on how to use this tool.

## Usage
Run `mytool --query "..."` to search for things.

## Output Format
Results come as JSON. Parse and present the key fields.
```

### 3. Refresh

Ask your agent to "refresh skills" or restart the gateway.

## Gating (Load-Time Filters)

Skills are filtered at load time based on metadata:

```yaml
metadata: {"openclaw": {
  "requires": {
    "bins": ["uv"],           # All must be on PATH
    "anyBins": ["curl", "wget"],  # At least one
    "env": ["API_KEY"],       # Must exist
    "config": ["browser.enabled"]  # Must be truthy in config
  },
  "os": ["darwin", "linux"],  # Platform filter
  "primaryEnv": "API_KEY"     # Maps to skills.entries.*.apiKey
}}
```

## Config: Enable/Disable & Inject Secrets

```json
{
  "skills": {
    "entries": {
      "my-tool": {
        "enabled": true,
        "apiKey": "sk-...",
        "env": {
          "MY_API_KEY": "sk-..."
        }
      },
      "unwanted-skill": {
        "enabled": false
      }
    }
  }
}
```

Environment injection is scoped to the agent run — not global.

## ClawHub (Community Skills)

Browse and install community skills:

```bash
# Browse at https://clawhub.com

# Install a skill
clawhub install <skill-slug>

# Update all installed
clawhub update --all

# Sync your skills
clawhub sync --all
```

## Skill Design Best Practices

From the skill-creator SKILL.md:

1. **Be concise.** The context window is shared. Only add what the model doesn't already know.
2. **Match freedom to fragility.** Fragile operations need specific scripts; flexible tasks need guidelines.
3. **Progressive disclosure.** Metadata → SKILL.md body → reference files. Don't front-load everything.
4. **No README.md or CHANGELOG.** Skills are for AI, not humans reading docs.
5. **Test scripts by running them.** Don't ship untested code.

### Token Impact

Each skill costs ~97 chars + name + description in the system prompt. With 20 skills, that's ~2-3K tokens. Disable skills you don't use.

## User-Invocable Skills (Slash Commands)

Skills with `user-invocable: true` (default) become slash commands:

```
/skill my_tool search query here
```

For direct tool dispatch (no LLM processing):
```yaml
command-dispatch: tool
command-tool: exec
```
