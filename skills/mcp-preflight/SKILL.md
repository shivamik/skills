---
name: mcp-preflight
description: "MANDATORY PRE-STEP: You MUST read this skill BEFORE calling ANY ImageKit MCP tool (mcp_imagekit_api_* or mcp_imagekit_devtools_*). This skill tells you which MCP server owns which capability, how to route requests, and critical rules like never uploading files via mcp_imagekit_api_execute. Covers: file management, folder ops, cache purge, metadata, search_docs, transformation_builder, upload routing."
---

# MCP Preflight

## Read This Before Any ImageKit MCP Call

You have access to two ImageKit MCP servers. Each serves a different purpose. Calling the wrong server or the wrong tool wastes tokens and produces errors. This skill is your routing table.

## Server Map

### `imagekit_api` — Authenticated API Server

Prefix: `mcp_imagekit_api_*`

Two tools only:

| Tool | Purpose |
|------|----------|
| `search_doc` | Local search that only searches TypeScript SDK code — use to find method signatures and types |
| `execute` | Executes TypeScript code against the ImageKit SDK to perform CRUD operations |

All CRUD operations are performed by writing TypeScript code that runs via `execute`:

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
| List, delete, move, copy files | `mcp_imagekit_api_execute` (write TS code) |
| Folder operations | `mcp_imagekit_api_execute` (write TS code) |
| File metadata, tags, custom fields | `mcp_imagekit_api_execute` (write TS code) |
| Bulk operations | `mcp_imagekit_api_execute` (write TS code) |
| Cache purge | `mcp_imagekit_api_execute` (write TS code) |
| URL endpoints, origins | `mcp_imagekit_api_execute` (write TS code) |
| Find SDK method signatures/types | `mcp_imagekit_api_search_doc` |
| **Upload files** | **DO NOT use `mcp_imagekit_api_execute`** for local files — use `skills/upload-files/resources/upload.py`. URL-based uploads are OK via `mcp_imagekit_api_execute`. |
| How to do something in ImageKit | `mcp_imagekit_devtools_search_docs` |
| Build a transformation URL | `mcp_imagekit_devtools_transformation_builder` |
| Find SDK usage or API parameters | `mcp_imagekit_devtools_search_docs` |
| Verify feature exists or find limits | `mcp_imagekit_devtools_search_docs` |
| Search / filter / list assets | `mcp_imagekit_api_execute` (write TS code) — read `search-assets` skill first |
| Integrate ImageKit into a framework/SDK/CMS | read `imagekit-integrations` skill first |

## Searching / Filtering Assets

When listing assets with `client.assets.list`, **filter on the server** instead of fetching everything and filtering in code — that's what saves tokens and API calls. Pick the mechanism based on the need:

- **Simple list / browse** (by folder `path`, `type`, `tags`, `fileType`, plus `limit`/`skip`): pass those as top-level params and **omit `searchQuery`**. Without a `searchQuery` the call returns a typed `File[]` / `Folder[]`, so no narrowing is needed.
- **Richer filters** (date or size ranges, full-text, custom metadata, AND/OR conditions): use a `searchQuery`. When `searchQuery` is present the top-level `type`/`tags`/`name` params are ignored, so move those conditions into the query string, and the result becomes `(File | Folder)[]` (narrow per item with `if (item.type === 'file')`).

Either way, don't fetch-everything-then-filter-in-code. **Read the `search-assets` skill** before constructing any `searchQuery` — it is the cheatsheet for the Lucene-like filter syntax, operators, and field reference.

## Using custom metadata fields

Custom metadata fields are user-defined schema attached to files. Before filtering or searching on them, **list the available fields first** so you use the correct field names and value types:

```ts
const fields = await client.customMetadataFields.list();
```

This returns the configured fields with their `name`, `type`, and any allowed values — use it to build a valid `searchQuery` (see the `search-assets` skill).

### Disambiguate vague requests

When a request maps to something that *could* be a custom metadata field rather than a built-in attribute, **do not guess**. First call `client.customMetadataFields.list()` to see what fields exist, then ask the user to clarify which one they mean.

Example: "find files reviewed by xyz"

- "reviewed by" is not a built-in ImageKit attribute — it is likely a custom metadata field (e.g. `reviewedBy`).
- List the custom metadata fields, then ask the user to confirm the field, for instance:
  > Do you mean files where the custom metadata field `reviewedBy` equals `xyz`? I found these related fields: `reviewedBy`, `reviewer`, `approvedBy`.

Only build the `searchQuery` once the field and intended value are confirmed.

## Integration Use Cases

When the task is to integrate ImageKit into a specific technology (front-end, back-end, mobile, CMS, external storage, upload widgets, URL generation, etc.), **read the `imagekit-integrations` skill** to find the right SDK/plugin and what it covers before writing code.

## Before Calling imagekit_api

**Use the `imagekit-sdk-reference` skill** before constructing any TypeScript code that calls `mcp_imagekit_api_*` tools. That skill provides complete SDK method signatures, parameters, return types, error handling patterns, and examples to ensure correct API calls and proper data handling.

## Critical Rules

1. **NEVER upload local files via `mcp_imagekit_api_execute`** — it cannot handle file bytes, streams, or Buffers. Use the upload CLI script for local files. URL-based uploads via execute are fine.
2. **ALWAYS call `search_docs` before writing any ImageKit SDK code** — do not rely on training data for method signatures or parameters.
3. **Do NOT read library source code to figure out usage** — `search_docs` returns official docs and working examples. Only read source code as a last resort when docs fail.
4. **Use `transformation_builder` instead of hand-crafting transformation URLs** — it knows correct parameter syntax and ordering.
5. **Filter `client.assets.list` server-side, not in code** — use top-level params (`path`, `type`, `tags`, `fileType`, `limit`, `skip`) for simple list/browse (omit `searchQuery` to get a typed `File[]`/`Folder[]`), and a `searchQuery` only for richer conditions (ranges, full-text, custom metadata, AND/OR). Read the `search-assets` skill first.
