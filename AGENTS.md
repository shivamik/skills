# AGENTS.md

## What Is This Repo?

This is a [skills.sh](https://skills.sh) skills repository for ImageKit. It contains reusable AI agent skills that teach coding agents how to use ImageKit's MCP servers, APIs, and tools correctly.

Skills are installed by users via:
```bash
npx skills add shivamik/skills
```

## `skills.sh.json`

The `skills.sh.json` file at the repo root controls how skills are displayed and categorized on the [skills.sh page](https://skills.sh/shivamik/skills). It groups skills into labeled sections.

### Structure

```json
{
  "$schema": "https://skills.sh/schemas/skills.sh.schema.json",
  "notGrouped": "bottom",
  "groupings": [
    {
      "title": "Category Name",
      "description": "Short description of this category.",
      "skills": ["skill-folder-name", "another-skill"]
    }
  ]
}
```

- `title` — Category heading shown on the page
- `description` — One-line summary of what skills in this group do
- `skills` — Array of skill folder names (must match folder names under `skills/`)
- `notGrouped` — Where ungrouped skills appear: `"top"` or `"bottom"`

### Categories

Use these categories when grouping skills:

| Category | Purpose |
|----------|---------|
| **General** | Skills that apply broadly — routing, preflight checks, documentation lookup |
| **MCP Tools** | Skills tied to specific MCP tool usage — transformation builder, upload, AI tasks |

Any skill not listed in a grouping appears in the "ungrouped" section (position controlled by `notGrouped`).

### Updating `skills.sh.json`

When adding or removing skills:

1. Add the skill's folder name to the appropriate `skills` array in `skills.sh.json`
2. If no existing category fits, create a new grouping
3. **Always ask the user to review the generated categories before committing** — auto-generated groupings may not match their intended organization
