---
name: imagekit-sdk-reference
description: "MANDATORY: Read this before writing any ImageKit SDK code or calling mcp_imagekit_api_* tools. Provides complete TypeScript method signatures, parameters, return types, error handling, and examples for the @imagekit/nodejs SDK."
---

# ImageKit TypeScript SDK Reference

Read this skill before calling any `mcp_imagekit_api_*` tool or writing TypeScript code against the ImageKit SDK. It contains exact method signatures, parameter types, return shapes, and error handling patterns for `@imagekit/nodejs`.

**Rules:**
1. Use exact parameter names — the SDK is strict about camelCase
2. `assets.list()` returns `(File | Folder)[]`. **Without** a `searchQuery`, pass `{ type: 'file' }` (or `'folder'`) to get a typed `File[]` (or `Folder[]`) directly — no narrowing needed. **With** a `searchQuery`, the API ignores the `type` param so the result stays a union: filter with `type = "file"` inside the query string and narrow with `for...of` + `if (item.type === 'file')`. Do NOT use `.filter((i): i is File => ...)` — the predicate collides with Deno's global `File` (see Gotchas).
3. In `execute`/MCP code, do NOT try/catch single API calls — the tool reports errors for you. Only catch when you branch on a specific failure, and **duck-type** the error (`'status' in err`) rather than `instanceof ImageKit.APIError`, since a value import of the SDK is not available in the sandbox.
4. Use `skip`/`limit` for pagination (max 1000 per request)
5. **Never upload files via `mcp_imagekit_api_execute`** — use the `upload-files` skill instead
6. Upload in SDK code is only valid when the `file` param is a **URL string** — never pass local file paths, Buffers, or streams through MCP
7. Nullable properties (`tags`, `AITags`, `customCoordinates`) require optional chaining (`?.`) or null checks
8. `.find()` returns `T | undefined` — always check for `undefined` before accessing properties

---

## TypeScript Gotchas

> **Key insight:** In the MCP execution context (Deno runtime), **always use `for...of` + `if` blocks** for type narrowing. The type predicate approach (`item is File`) fails because Deno's global `File` (the Blob-based Web API) shadows the ImageKit SDK's `File` type.

| Pattern | Problem | Fix |
|---------|---------|-----|
| `.filter(i => i.type === 'file')` | Does NOT narrow the union type | Use `for...of` + `if` |
| `.filter((i): i is File => ...)` | Global `File` shadows SDK `File` in Deno | Use `for...of` + `if` |
| `.find(i => ...)` | Returns `T \| undefined` | Check for `undefined` + narrow type |
| `.filter().map()` | `.map()` inherits un-narrowed type | Use `for...of` + `if` + push |
| `file.tags` | `string[] \| null` | Use `?.` or null check |
| `file.AITags` | `Array \| null` | Use `?.` or null check |
| `for...of` + `if` block | — | **Always works — use this** |

### Why `.filter()` fails in MCP context (two separate issues)

**Issue 1: `.filter()` without a type predicate does not narrow.**
TypeScript's `.filter()` returns the same array type unless the callback has a type predicate. A boolean callback does NOT narrow the output type.

```typescript
// ❌ DOES NOT COMPILE — no narrowing:
const files = result.filter((item) => item.type === 'file');
files[0].fileId; // TS ERROR: 'fileId' does not exist on type 'File | Folder'
```

**Issue 2: Type predicate `item is File` collides with global `File` in Deno.**
In the MCP execution environment (Deno), the global `File` type (Web API Blob-based) shadows the SDK's `File` type. TypeScript resolves `File` in the predicate to the global, causing:

```typescript
// ❌ DOES NOT COMPILE in Deno/MCP — global File ≠ SDK File:
const files = result.filter((item): item is File => item.type === 'file');
// Error: Type 'File' is not assignable to type 'import("@imagekit/nodejs/...").File'
//   Types of property 'type' are incompatible.
//   Type 'string' is not assignable to type '"file" | "file-version"'
```

**✅ RECOMMENDED: use `for...of` + `if` whenever the result is a union** (i.e. when you used a `searchQuery`, `type: 'all'`, or no `type`):
```typescript
// ✅ ALWAYS WORKS — control flow narrowing, no type name needed:
const result = await client.assets.list({ searchQuery: 'type = "file"', limit: 10 });
const files = [];
for (const item of result) {
  if (item.type === 'file') {
    files.push({
      name: item.name,
      fileId: item.fileId,
      filePath: item.filePath,
      size: item.size,
      url: item.url
    });
  }
}
return files;
```

### `.find()` returns `T | undefined`

```typescript
const item = assets.find((i) => i.name === 'hero.jpg');
// Type: (File | Folder) | undefined

item.fileId; // ❌ Two errors: possibly undefined AND possibly Folder

// ✅ Fix:
if (item && item.type === 'file') {
  item.fileId; // works
}
```

### Nullable properties on File

```typescript
const file = await client.files.get(fileId);

file.tags.length;     // ❌ tags is string[] | null
file.tags?.length;    // ✅ optional chaining

file.AITags.map(...); // ❌ AITags is Array | null
file.AITags?.map(...); // ✅
```

---

## Types

### File
```typescript
{
  fileId: string; name: string; filePath: string; type: 'file' | 'file-version';
  url: string; thumbnail: string; isPrivateFile: boolean; isPublished: boolean;
  // Media
  fileType: string; // 'image' | 'non-image'
  mime: string; size: number; width: number; height: number; hasAlpha: boolean;
  // Video-only
  duration: number; videoCodec: string; audioCodec: string; bitRate: number;
  // Tags & metadata
  tags: string[] | null;
  AITags: Array<{ name: string; confidence: number; source: string }> | null;
  customMetadata: Record<string, unknown>; description: string;
  customCoordinates: string | null;
  embeddedMetadata: Record<string, unknown>;
  // Versioning
  versionInfo: { id: string; name: string };
  createdAt: string; updatedAt: string;
}
```

### Folder
```typescript
{
  folderId: string; folderPath: string; name: string; type: 'folder';
  customMetadata: Record<string, unknown>;
  createdAt: string; updatedAt: string;
}
```

### CustomMetadataField
```typescript
{
  id: string; name: string; label: string;
  schema: { type: 'Text' | 'Number' | 'Date' | 'Boolean' | 'SingleSelect' | 'MultiSelect'; /* ... */ };
}
```

---

## File Operations

### Upload a file (URL only via MCP)
```typescript
// ⚠️ Via mcp_imagekit_api_execute, ONLY URL-based uploads work.
// For local files, use the upload-files skill (CLI script) instead.
const file = await client.files.upload({
  file: 'https://example.com/img.jpg', // URL string ONLY in MCP context
  fileName: 'img.jpg',
  folder: '/uploads',
  tags: ['tag1'],
  customMetadata: { key: 'value' },
  // Key optional params:
  // useUniqueFileName: true,      // default true — appends random suffix
  // isPrivateFile: false,
  // overwriteFile: false,         // replace existing file at same path
  // overwriteTags: false,
  // overwriteCustomMetadata: false,
  // extensions: [{ name: 'google-auto-tagging', maxTags: 5 }],
  // transformation: { pre: 'w-200' },
  // webhookUrl: 'https://...',
});
// Returns: File object
```

### Get file details
```typescript
const file = await client.files.get(fileId); // Returns: File
```

### List / search assets
```typescript
const result = await client.assets.list({
  searchQuery: 'name = "img.jpg"', // Lucene-like syntax
  path: '/uploads',
  fileType: 'image',           // 'image' | 'non-image' | 'all'
  type: 'file',                // 'file' | 'folder' | 'file-version' | 'all'
  sort: 'ASC_NAME',
  skip: 0,
  limit: 100,
});
// Returns: (File | Folder)[] — a flat array, NOT { files, folders }
```

**Type narrowing depends on whether you use `searchQuery`:**

- **Without `searchQuery`** — pass `{ type: 'file' }` (or `'folder'`); the return type is `File[]` (or `Folder[]`), no narrowing needed:
  ```typescript
  const files = await client.assets.list({ type: 'file', path: '/uploads', limit: 100 }); // File[]
  files[0].fileId; // ✅ no narrowing required
  ```
- **With `searchQuery`** — the API ignores the `type` param, so the result is `(File | Folder)[]`. Filter with `type = "file"` inside the query and narrow with `for...of` + `if`:
  ```typescript
  const result = await client.assets.list({ searchQuery: 'type = "file" AND size > "2mb"', limit: 100 });
  const files = [];
  for (const item of result) {
    if (item.type === 'file') {
      files.push({ name: item.name, fileId: item.fileId, filePath: item.filePath, size: item.size, url: item.url });
    }
  }
  return files;
  ```

Do NOT narrow a union with `.filter((item): item is File => ...)` — in Deno the global `File` shadows the SDK's `File` and the predicate fails to compile. Use `for...of` + `if`.

**Shared properties** (safe on both File and Folder): `name`, `type`, `customMetadata`, `createdAt`, `updatedAt`

**File-only properties** (require narrowing): `fileId`, `filePath`, `fileType`, `mime`, `size`, `width`, `height`, `url`, `thumbnail`, `tags`, `AITags`, `description`, `isPrivateFile`, `isPublished`, `customCoordinates`, `embeddedMetadata`, `versionInfo`, `duration`, `videoCodec`, `audioCodec`, `bitRate`, `hasAlpha`

**Folder-only properties**: `folderId`, `folderPath`

### Update / delete file
```typescript
await client.files.update(fileId, { tags: ['newTag'], customMetadata: { key: 'value' } }); // Returns: File
await client.files.delete(fileId); // Returns: void
```

### Copy / move / rename file
```typescript
await client.files.copy({ sourceFilePath: '/a/img.jpg', destinationPath: '/b/', includeFileVersions: false });
await client.files.move({ sourceFilePath: '/a/img.jpg', destinationPath: '/b/' });
await client.files.rename({ filePath: '/a/img.jpg', newFileName: 'new.jpg', purgeCache: false });
```

---

## Bulk Operations

```typescript
await client.files.bulk.delete({ fileIds: ['id1', 'id2'] }); // { successfullyDeletedFileIds }
await client.files.bulk.addTags({ fileIds: ['id1'], tags: ['promo'] });    // max 50 files
await client.files.bulk.removeTags({ fileIds: ['id1'], tags: ['old'] });
await client.files.bulk.removeAITags({ fileIds: ['id1'], AITags: ['cat'] });
```

---

## Folder Operations

```typescript
await client.folders.create({ folderName: 'myfolder', parentFolderPath: '/' });
await client.folders.delete({ folderPath: '/myfolder' });

// Copy / move / rename — async operations, return { jobId }
const { jobId } = await client.folders.copy({ sourceFolderPath: '/a', destinationPath: '/b/' });
const { jobId } = await client.folders.move({ sourceFolderPath: '/a', destinationPath: '/b/' });
const { jobId } = await client.folders.rename({ folderPath: '/a', newFolderName: 'renamed' });

// Check job status
const job = await client.folders.job.get(jobId);
// job.status: 'Pending' | 'Completed'
```

---

## File Versions

```typescript
const versions = await client.files.versions.list(fileId);      // Returns: File[]
const version = await client.files.versions.get(versionId, { fileId }); // Returns: File
await client.files.versions.restore(versionId, { fileId });     // Returns: File
await client.files.versions.delete(versionId, { fileId });      // Returns: void
```

---

## File Metadata

```typescript
const metadata = await client.files.metadata.getFromURL({ url: 'https://ik.imagekit.io/x/img.jpg' });
// Returns EXIF/IPTC metadata: { height, width, exif, iptc, xmp, ... }
```

---

## Cache Invalidation

```typescript
const purge = await client.cache.invalidation.create({ url: 'https://ik.imagekit.io/x/img.jpg' });
const status = await client.cache.invalidation.get(purge.requestId);
// status.status: 'Pending' | 'Completed'
```

---

## URL Building

```typescript
const url = client.helper.buildSrc({
  urlEndpoint: 'https://ik.imagekit.io/your_id',
  src: '/path/img.jpg',
  transformation: [{ width: 400, height: 300, format: 'webp', quality: 80 }],
  signed: true,
  expiresIn: 3600,
});
// Returns: string
```

---

## Custom Metadata Fields

```typescript
// Define schema fields for your media library
const field = await client.customMetadataFields.create({
  name: 'brand', label: 'Brand Name',
  schema: { type: 'Text', defaultValue: '', isValueRequired: false },
});
const fields = await client.customMetadataFields.list(); // Returns: CustomMetadataField[]
await client.customMetadataFields.update(field.id, { label: 'Updated Label' });
await client.customMetadataFields.delete(field.id);
```

---

## Account Usage

```typescript
const usage = await client.accounts.usage.get({ startDate: '2025-01-01', endDate: '2025-01-31' });
// { bandwidthBytes, mediaLibraryStorageBytes, extensionUnitsCount, videoProcessingUnitsCount }
```

---

## Pagination & Error Handling

```typescript
// Offset-based pagination — skip / limit (max 1000). No cursor support.
for (let skip = 0; ; skip += 100) {
  const page = await client.assets.list({ skip, limit: 100 });
  if (!page.length) break;
  for (const item of page) {
    if (item.type === 'file') {
      // item is narrowed to File here
    }
  }
}

// Error handling — in execute/MCP code you normally DON'T need try/catch: let the
// error propagate and the tool reports it for you. The ONLY reason to catch is to
// branch on a specific failure (e.g. treat 404 as "not found") — and then you must
// re-throw everything you don't handle. Duck-type the error: a value import of the
// SDK (import ImageKit from '@imagekit/nodejs') is NOT available in the Deno sandbox
// and throws at runtime, so don't rely on `instanceof ImageKit.APIError`.
try {
  return await client.files.get(fileId);
} catch (err) {
  if (err && typeof err === 'object' && 'status' in err) {
    const e = err as { status?: number; message?: string };
    if (e.status === 404) return null;  // the one case we handle
  }
  throw err;  // re-throw anything else — don't swallow it
}
// Auto-retries: connection errors, 408/409/429/5xx — up to 2× with exponential backoff
```
