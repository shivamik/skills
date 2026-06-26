---
name: search-assets
description: "MANDATORY: Read this whenever calling client.assets.list() with a searchQuery, or when searching/filtering/listing ImageKit files or folders. Covers the Lucene-like searchQuery filter syntax, operators, field reference, and examples for filtering by name, tags, date, size, format, path, or custom/embedded metadata."
---

# ImageKit Search Queries (`searchQuery`)

`client.assets.list({ searchQuery })` takes a Lucene-like filter string. A good query is the difference between one precise API call and paging through everything. This skill is the cheatsheet for building that string.

## Syntax

- **Operators:** `=` `:` `<` `<=` `>` `>=` `IN` `NOT IN` `NOT =` `HAS` `EXISTS` `NOT EXISTS`
- **Combine:** `AND`, `OR`. **Group:** `( ... )`
- **Quoting:** string values in `"double quotes"`; numbers and booleans (`true`/`false`) unquoted; `customMetadata`/`embeddedMetadata` field names must be quoted, e.g. `"customMetadata.brand"`.
- `:` = begins-with / within-subfolders. `=` = exact. `HAS` = case-insensitive full-text (tokenized).

## Field reference

| Field | Operators | Value notes |
|-------|-----------|-------------|
| `name` | `=` `:` `IN` `NOT =` `NOT IN` `HAS` | String. `=` exact (case-sensitive), `:` begins-with, `HAS` case-insensitive full-text |
| `tags` | `IN` `NOT IN` `HAS` `EXISTS` `NOT EXISTS` | Array. Matches both `tags` and `AITags` |
| `type` | `=` `IN` `NOT =` `NOT IN` | `"file"` \| `"file-version"` \| `"folder"` |
| `id` | `=` `IN` `NOT =` `NOT IN` | fileId or folderId string |
| `createdAt` / `updatedAt` | `=` `<` `<=` `>` `>=` `IN` `NOT =` `NOT IN` | ISO 8601 (`"2020-01-01"`, `"2020-01-01T12:12:12"`) or relative (`"1h"` `"2d"` `"3w"` `"4m"` `"1y"`) |
| `width` / `height` | `=` `<` `<=` `>` `>=` `IN` `NOT =` `NOT IN` | Numeric px. Images only |
| `size` | `=` `<` `<=` `>` `>=` `IN` `NOT =` `NOT IN` | Bytes (`1024`) or string (`"1mb"`, `"10kb"`) |
| `format` | `=` `IN` | `"jpg"` `"png"` `"webp"` `"gif"` `"svg"` `"avif"` `"pdf"` `"mp4"` … |
| `private` / `published` / `transparency` | `=` | Boolean, unquoted: `true` / `false` |
| `createdBy` | `=` `IN` `NOT =` `NOT IN` | Uploader email string |
| `path` | `=` `:` `IN` `NOT =` `NOT IN` | `=` exact folder only; `:` folder + subfolders |
| `"customMetadata.<field>"` | type-dependent (`=` `:` `IN` `HAS` `<` `>` `EXISTS` …) | Quote the field name. Operators follow the field's schema type |
| `"embeddedMetadata.<field>"` | type-dependent | Quote the field name. `Keywords` uses `IN`/`EXISTS`; `DateTimeOriginal` uses date ops; `LocationTaken` supports geo (`"40,100 5km"`) |

## Query examples

```text
name = "red-dress.jpg"                                  exact name (case-sensitive)
name : "red-dress"                                      name begins with red-dress
name HAS "red dress"                                    full-text: contains red AND dress (any case)
id = "64fb1c2d8a7b4c1234567890"                         single file/folder by id

createdAt > "7d"                                        uploaded in last 7 days
createdAt < 2020-01-01                                  uploaded before Jan 1 2020 (00:00 UTC)
updatedAt >= "2024-06-01T00:00:00"                      modified on/after a timestamp
createdAt > "7d" AND size > "2mb"                       recent AND large

size <= "1mb"                                           1MB or smaller
width > 500 AND height > 500                            at least 500x500 (images)
format = "png"                                          PNG files
format IN ["jpg", "webp"]                               JPG or WebP

tags IN ["sale", "summer"]                              has sale OR summer (tags or AITags)
tags NOT IN ["draft"]                                   excludes draft
tags HAS "sale"                                         full-text token match (sales, summer-sale…)
tags EXISTS                                             has at least one tag
tags NOT EXISTS                                         untagged files

type = "file"                                           files only
private = true                                          private files
published = false                                       drafts / unpublished
transparency = true                                     images with an alpha layer

path = "/sales-banner/"                                 exactly this folder, no subfolders
path : "/sales-banner/"                                 this folder AND its subfolders
format = "png" AND path : "/sales-banner/"              PNGs under a folder tree

"customMetadata.category" IN ["clothing", "accessories"]
"customMetadata.rating" > 4.3
"customMetadata.active" = true
"customMetadata.description" HAS "red"
"embeddedMetadata.DateTimeOriginal" > "1y"
"embeddedMetadata.LocationTaken" = "40,100 5km"

(size < "1mb" AND width > 500) OR (tags IN ["summer-sale", "banner"])
```

## Custom metadata — discover field names first

`"customMetadata.<field>"` only matches fields defined in your media library's custom-metadata schema, and the **allowed operators depend on the field's type** (Text/Textarea → `=` `:` `IN` `HAS`; Number/Date → `<` `<=` `>` `>=`; Boolean → `=`; Single/MultiSelect → `IN`). If the user's request mentions a custom attribute (brand, category, rating, SKU, season…), **list the schema first** to get the exact field name and type, then build the query:

```typescript
const fields = await client.customMetadataFields.list();
// each: { id, name, label, schema: { type: 'Text' | 'Number' | 'Date' | 'Boolean' | 'SingleSelect' | 'MultiSelect', ... } }
const brand = fields.find((f) => f.name === 'brand' || f.label === 'Brand');
if (!brand) throw new Error('No "brand" custom metadata field is defined');

const result = await client.assets.list({
  searchQuery: 'type = "file" AND "customMetadata.brand" = "nike"',
  limit: 100,
});
```

Quote the whole `"customMetadata.<field>"` token. `"embeddedMetadata.<field>"` (e.g. `Keywords`, `DateTimeOriginal`, `LocationTaken`) works the same way, but embedded fields are read from the file itself and are not user-defined — no schema lookup needed.

## TypeScript usage

`client.assets.list()` returns `(File | Folder)[]`. How you get a typed result depends on whether you use `searchQuery`:

**No `searchQuery` — pass `type` to get a typed array directly (no narrowing):**
```typescript
const files = await client.assets.list({ type: 'file', path: '/uploads', limit: 100 });
//    ^ File[] — fileId, size, url are all available without narrowing
```

**With `searchQuery` — the API IGNORES the `type`/`tags`/`name` params**, so the result stays the `(File | Folder)[]` union. Put the type filter *inside* the query string and narrow each item:
```typescript
const result = await client.assets.list({
  searchQuery: 'type = "file" AND createdAt >= "7d" AND size > "2mb"',
  limit: 100,
});
const files = [];
for (const item of result) {
  if (item.type === 'file') {
    files.push({ name: item.name, fileId: item.fileId, size: item.size });
  }
}
```

**Full-text search — always pair `HAS` with `DESC_RELEVANCE`:**
```typescript
const result = await client.assets.list({
  searchQuery: 'name HAS "red dress"',
  sort: 'DESC_RELEVANCE',
  limit: 20,
});
```

**Paginate with `skip`/`limit` (max 1000 per page):**
```typescript
for (let skip = 0; ; skip += 100) {
  const page = await client.assets.list({
    searchQuery: 'type = "file" AND tags IN ["sale", "summer"]',
    skip,
    limit: 100,
  });
  if (!page.length) break;
  for (const item of page) {
    if (item.type === 'file') {
      // item narrowed to File
    }
  }
}
```

## Gotchas

- **`searchQuery` overrides params.** When `searchQuery` is present, the top-level `type`, `tags`, and `name` params have NO effect — express those filters inside the query string instead (e.g. `type = "file"`).
- `name` `=`/`:` are **case-sensitive**; use `HAS` for case-insensitive matching.
- `tags` queries search **both** `tags` and `AITags`.
- `HAS` tokenizes on spaces/punctuation; the last token matches as a prefix (`"red"` matches `redwoods`). Pair with `sort: 'DESC_RELEVANCE'`.
- Booleans (`private`, `published`, `transparency`) are unquoted; everything string-valued is quoted.
- `width`/`height`/`transparency` apply to images only.
