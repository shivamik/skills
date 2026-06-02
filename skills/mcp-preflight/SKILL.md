---
name: mcp-preflight
description: "MANDATORY PRE-STEP: You MUST read this skill BEFORE calling ANY ImageKit MCP tool (mcp_imagekit_api_* or mcp_imagekit_devtools_*). This skill tells you which MCP server owns which capability, how to route requests, and critical rules like never using mcp_imagekit_api_upload_files. Covers: file management, folder ops, cache purge, metadata, search_docs, transformation_builder, upload routing."
---

# MCP Preflight

## Read This Before Any ImageKit MCP Call

You have access to two ImageKit MCP servers. Each serves a different purpose. Calling the wrong server or the wrong tool wastes tokens and produces errors. This skill is your routing table.

## Server Map

### `imagekit_api` — Authenticated API Server

Prefix: `mcp_imagekit_api_*`

Handles all CRUD operations on your ImageKit media library:

- List, search, get details of files
- Delete, move, copy, rename files
- Create, delete, move, copy folders
- Bulk tag/untag operations
- File versions (list, restore, delete)
- Custom metadata fields (CRUD)
- Cache invalidation (purge)
- URL endpoints and origins management
- Account usage stats

### `imagekit_devtools` — Public Tools Server

Prefix: `mcp_imagekit_devtools_*`

Two tools only — no auth required:

| Tool | Purpose |
|------|---------|
| `search_docs` | RAG-powered search across ImageKit docs, guides, API refs, SDKs, community posts |
| `transformation_builder` | Builds transformation URLs from natural language descriptions |

## Routing Table

| Need | Route To |
|------|----------|
| List, delete, move, copy files | `mcp_imagekit_api_*` |
| Folder operations | `mcp_imagekit_api_*` |
| File metadata, tags, custom fields | `mcp_imagekit_api_*` |
| Bulk operations | `mcp_imagekit_api_*` |
| Cache purge | `mcp_imagekit_api_*` |
| URL endpoints, origins | `mcp_imagekit_api_*` |
| **Upload files** | **DO NOT use MCP** — use `skills/upload-files/resources/upload.py` |
| How to do something in ImageKit | `mcp_imagekit_devtools_search_docs` |
| Build a transformation URL | `mcp_imagekit_devtools_transformation_builder` |
| Find SDK usage or API parameters | `mcp_imagekit_devtools_search_docs` |
| Verify feature exists or find limits | `mcp_imagekit_devtools_search_docs` |

## Critical Rules

1. **NEVER use `mcp_imagekit_api_upload_files`** — it cannot handle local file bytes. Always use the upload CLI script instead.
2. **ALWAYS call `search_docs` before writing any ImageKit SDK code** — do not rely on training data for method signatures or parameters.
3. **Do NOT read library source code to figure out usage** — `search_docs` returns official docs and working examples. Only read source code as a last resort when docs fail.
4. **Use `transformation_builder` instead of hand-crafting transformation URLs** — it knows correct parameter syntax and ordering.
