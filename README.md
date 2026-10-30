# my-skills

Shared agent skills. Each skill is one folder and does not depend on the others.

```
skills/<name>/SKILL.md
```

`<name>` is the skill id. It matches the folder and the `name` field in `SKILL.md`.

## Install

From the project that should load the skill:

```bash
npx skills add MohammedAbidNafi/my-skills --skill <name>
```

Or copy the folder:

```
skills/<name>  →  .cursor/skills/<name>
```

## Add a skill

1. Add `skills/<name>/SKILL.md`.
2. Append one block to the catalog, using the shape below.

```markdown
### <name>

One sentence on what the skill is for.

- Instructions: [SKILL.md](skills/<name>/SKILL.md)
- Install: `npx skills add MohammedAbidNafi/my-skills --skill <name>`
```

## Catalog

### figma-mcp-cache

Read Figma files through the cache-enabled Figma Context MCP (`@pactortester/figma-mcp-cached`) so repeat reads stay local.

- Instructions: [SKILL.md](skills/figma-mcp-cache/SKILL.md)
- Install: `npx skills add MohammedAbidNafi/my-skills --skill figma-mcp-cache`
