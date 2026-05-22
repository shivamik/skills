# ImageKit Skills

[![skills.sh](https://skills.sh/b/shivamik/skills)](https://skills.sh/shivamik/skills)

Reusable AI agent skills for [ImageKit.io](https://imagekit.io) — install them with the `skills` CLI to enhance your coding agent's capabilities.

## Installation

```bash
npx skills add shivamik/skills --all
```

## Skills

| Skill | Description |
|-------|-------------|
| **mcp-preflight** | Mandatory routing guide — tells the agent which MCP server to call for what, before every ImageKit tool invocation |
| **search-docs** | Search ImageKit documentation with optimized queries and source selection |
| **transformation-builder** | Build ImageKit image/video transformations — AI editing, background removal, resize, crop, overlays, and more |
| **upload-files** | Upload files to ImageKit media library with folder paths, tags, and metadata |

## Usage

Once installed, these skills are automatically available to your AI agent. The agent will consult the relevant skill before performing ImageKit operations, ensuring correct tool usage and better results.

## License

MIT
