# opencanvas skills

Installable agent skills for [opencanvas](https://getopencanvas.dev) — the review surface for agent-built prototypes.

Each skill is a `SKILL.md` that teaches your coding agent how to author and iterate prototypes over the opencanvas MCP, then act on human comments in the review loop.

## Skills

- **[opencanvas-prototype](./opencanvas-prototype/SKILL.md)** — build and iterate mobile & desktop prototypes for human review over the opencanvas MCP.

## Install

Cross-agent (Claude Code, Codex, Cursor, opencode, Cline, Copilot, Windsurf, and 60+ more) via the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add bjoern2000/opencanvas-skills
```

Or just one skill:

```bash
npx skills add bjoern2000/opencanvas-skills --skill opencanvas-prototype
```

Claude Code, manually:

```bash
curl -o ~/.claude/skills/opencanvas-prototype/SKILL.md \
  https://raw.githubusercontent.com/bjoern2000/opencanvas-skills/main/opencanvas-prototype/SKILL.md
```

## Connect the MCP first

The skill drives the opencanvas MCP server — add it before using the skill:

```bash
claude mcp add --transport http opencanvas https://app.getopencanvas.dev/mcp
```

Other MCP clients — add to your config:

```json
{ "opencanvas": { "type": "http", "url": "https://app.getopencanvas.dev/mcp" } }
```

## Links

- Product: [getopencanvas.dev](https://getopencanvas.dev)
- App: [app.getopencanvas.dev](https://app.getopencanvas.dev)
