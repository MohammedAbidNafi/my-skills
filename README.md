# my-skills

Skills for Cursor and other agents that read `SKILL.md`.

Each skill is a folder under `skills/` with a `SKILL.md` file.

## Install one skill

From a project that should use a skill:

```bash
npx skills add <owner>/my-skills --skill figma-mcp-cache
```

Or copy `skills/<name>` into `.cursor/skills/<name>` in that project.

## Skills

| Skill | What it does |
| --- | --- |
| [figma-mcp-cache](skills/figma-mcp-cache/SKILL.md) | Read Figma files through the cache-enabled Figma Context MCP so repeat reads stay local. |
