---
name: imagekit-sdk-reference
description: "MANDATORY: Read this before writing any ImageKit SDK code or calling mcp_imagekit_api_* tools. Provides complete TypeScript method signatures, parameters, return types, error handling, and examples for the @imagekit/nodejs SDK."
---

# ImageKit TypeScript SDK Reference

Read this skill before calling any `mcp_imagekit_api_*` tool or writing TypeScript code against the ImageKit SDK. It contains exact method signatures, parameter types, return shapes, and error handling patterns for `@imagekit/nodejs`.

**Rules:**
1. Use exact parameter names — the SDK is strict about camelCase
2. `assets.list()` returns `(File | Folder)[]` — always narrow the type before accessing file-specific properties
3. Always wrap calls in try/catch for `ImageKit.APIError`
4. Use `skip`/`limit` for pagination (max 1000 per request)
5. **Never upload files via `mcp_imagekit_api_execute`** — use the `upload-files` skill instead
6. Upload in SDK code is only valid when the `file` param is a **URL string** — never pass local file paths, Buffers, or streams through MCP

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

**⚠️ CRITICAL: Type narrowing is required.** Even with `type: 'file'`, the TypeScript return type is `(File | Folder)[]`. You MUST narrow before accessing file-only properties like `filePath`, `fileType`, `size`, `url`:

```typescript
// ✅ CORRECT — narrow the type first
const result = await client.assets.list({ type: 'file', limit: 10 });
const files = result.filter((item): item is File => item.type === 'file');
files.forEach(f => console.log(f.name, f.filePath, f.size, f.url)); // No TS error

// ❌ WRONG — result.forEach(f => f.filePath) → TS error: 'filePath' not on Folder
```

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

## Webhooks

```typescript
// Verified parse (requires webhookSecret on client)
const event = client.webhooks.unwrap(rawBody, { headers: req.headers });
// event.type: 'FILE.CREATE' | 'FILE.UPDATE' | 'FILE.DELETE'
//   | 'UPLOAD.POST_TRANSFORM.SUCCESS' | 'UPLOAD.POST_TRANSFORM.ERROR'
//   | 'VIDEO_TRANSFORMATION.ACCEPTED' | 'VIDEO_TRANSFORMATION.READY' | 'VIDEO_TRANSFORMATION.ERROR'

// Unverified parse
const event = client.webhooks.unsafeUnwrap(rawBody);
```

---

## Pagination & Error Handling

```typescript
// Offset-based pagination — skip / limit (max 1000). No cursor support.
for (let skip = 0; ; skip += 100) {
  const page = await client.assets.list({ skip, limit: 100 });
  if (!page.length) break;
  const files = page.filter((item): item is File => item.type === 'file');
}

// Error handling
import ImageKit from '@imagekit/nodejs';
try { await client.files.get('bad-id'); }
catch (err) {
  if (err instanceof ImageKit.APIError) {
    err.status;  // 400 | 401 | 403 | 404 | 409 | 422 | 429 | 5xx
    err.message; err.error; err.headers;
  }
}
// Auto-retries: connection errors, 408/409/429/5xx — up to 2× with exponential backoff
```
