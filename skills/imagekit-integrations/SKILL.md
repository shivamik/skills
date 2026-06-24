---
name: imagekit-integrations
description: "Index of ImageKit SDKs, plugins, and integrations — front-end, back-end, mobile, CMS, external storage, video player, media library & upload widgets, AI/automation, and URL generation. Use to find the right integration for a technology and what it covers, then query search_docs for implementation details."
---

# ImageKit Integrations

ImageKit ships SDKs (front-end, back-end, mobile), no-code plugins (CMS, e-commerce), external-storage connectors, a video player, embeddable widgets, AI/automation tooling, and direct URL-generation helpers. This skill is an index of all of them and what each covers.

## When to use this skill

- You're integrating ImageKit into an app and need to know **which SDK, plugin, player, or widget exists** for your stack.
- You want a quick sense of **what a given integration can do** before committing.
- You need to route a "how do I integrate ImageKit with X" question to the right place.

## How to use this skill

1. Skim the sections below — they're grouped by integration type (SDKs, plugins, storage, player, widgets, automation, URL generation). Find the one that matches your stack or use case and read its summary to confirm it fits.
2. For implementation details, **query [`mcp_imagekit_devtools_search_docs`](https://imagekit.io/docs/build-with-ai)** with the `imagekit_guides` and `imagekit_sdk` sources — e.g. *"generate signed URLs in Node.js"* or *"responsive srcset in React"*. This returns only the relevant snippets and is far cheaper and faster than loading a whole page.
3. Only fetch a full doc page (append `.md` to its URL for plain text) when you genuinely need the **entire** setup guide end-to-end. Prefer `search_docs` for everything else.

## SDKs

### Front-End

For web apps that generate optimized URLs and upload from the browser. Covers:

- Loading optimized images & videos with automatic format/quality
- Resize, crop, and chained transformations (multiple ops in one request)
- Text, image, video, and subtitle overlays
- Responsive images (`srcset` / `sizes`), lazy loading, and low-quality placeholders (LQIP)
- Client-side upload with tagging and metadata

All front-end SDKs wrap the [JavaScript SDK](https://imagekit.io/docs/integration/javascript) — drop down to it directly when you need full control.

| Tech | Guide |
|------|-------|
| [JavaScript](https://imagekit.io/docs/integration/javascript) | Vanilla / browser |
| [React](https://imagekit.io/docs/integration/react) | React components & hooks |
| [Next.js](https://imagekit.io/docs/integration/nextjs) | App & Pages router |
| [Vue.js](https://imagekit.io/docs/integration/vuejs) | Vue components |
| [Angular](https://imagekit.io/docs/integration/angular) | Angular components |
| [Astro](https://imagekit.io/docs/integration/astro) | Astro components |
| [React Native](https://imagekit.io/docs/integration/react-native) | Mobile web/native |

### Back-End

For servers that build URLs, secure delivery, and manage the media library. Covers:

- URL construction with transformations
- **Signed URLs** for secure, time-limited delivery of private media
- Generating auth params (`token`, `expire`, `signature`) so the client can upload securely
- File upload, tagging, and metadata operations
- Developer-friendly wrappers over the media management APIs (list, search, move, delete, versions, etc.)

| Tech | Guide |
|------|-------|
| [Node.js](https://github.com/imagekit-developer/imagekit-nodejs) | JS/TS runtime |
| [Python](https://imagekit.io/docs/integration/python) | Python SDK |
| [Java](https://imagekit.io/docs/integration/java) | Java SDK |
| [PHP](https://imagekit.io/docs/integration/php) | PHP SDK |
| [Ruby](https://imagekit.io/docs/integration/ruby) | Ruby (+ Rails, CarrierWave) |
| [.NET](https://github.com/imagekit-developer/imagekit-dotnet) | .NET SDK |
| [Go](https://github.com/imagekit-developer/imagekit-go) | Go SDK |

### Mobile

Native SDKs for iOS and Android. Covers:

- URL construction for optimized images & videos, including adaptive bitrate (ABR) video streaming
- Media loading — the Android SDK integrates with Glide, Picasso, Coil, and Fresco
- Responsive image loading
- Upload with an upload policy (network type, retry behavior, etc.)
- Pre-upload preprocessing to control file size and dimensions

The iOS SDK currently trails Android on features.

| Tech | Guide |
|------|-------|
| [Android](https://github.com/imagekit-developer/imagekit-android) | Glide/Picasso/Coil/Fresco |
| [iOS](https://github.com/imagekit-developer/imagekit-ios) | Swift SDK |

## No-code Plugins & Storage

### CMS & E-commerce

Native plugins that route media through ImageKit from your existing platform — automatic optimization, responsive delivery, and CDN caching without writing integration code.

| Platform | Guide |
|----------|-------|
| [WordPress](https://imagekit.io/docs/integration/wordpress) | WP plugin |
| [Shopify](https://imagekit.io/docs/integration/shopify) | Shopify app |
| [Magento](https://imagekit.io/docs/integration/magento) | Magento extension |
| [Strapi](https://imagekit.io/docs/integration/strapi) | Strapi provider |
| [Contentful](https://imagekit.io/docs/integration/contentful) | Contentful app |
| [Sanity](https://imagekit.io/docs/integration/sanity) | Sanity plugin |
| [Netlify](https://imagekit.io/docs/integration/netlify) | Netlify integration |

### External Storage (origins)

Connect your existing storage as an ImageKit origin so it optimizes and delivers files already there — no re-uploading or migration needed.

| Provider | Guide |
|----------|-------|
| [AWS S3](https://imagekit.io/docs/integration/aws-s3) | S3 bucket origin |
| [Google Cloud Storage](https://imagekit.io/docs/integration/google-cloud-storage) | GCS bucket origin |
| [Azure Blob Storage](https://imagekit.io/docs/integration/azure-blob-storage) | Azure container origin |
| [Firebase Storage](https://imagekit.io/docs/integration/firebase-storage) | Firebase origin |
| [DigitalOcean Spaces](https://imagekit.io/docs/integration/digitalocean-spaces) | Spaces origin |
| [Web server / proxy](https://imagekit.io/docs/integration/web-server) | Any HTTP origin |

## Player & Widgets

### Video Player SDK

A drop-in [video player](https://imagekit.io/docs/video-player/overview) built on Video.js with ImageKit-powered features. Covers:

- Adaptive bitrate streaming and seek-preview thumbnails
- AI-generated [subtitles & chapters](https://imagekit.io/docs/video-player/subtitles-and-chapters) (speech-to-text in 50+ languages, word-level highlighting)
- [Shoppable videos](https://imagekit.io/docs/video-player/shoppable-videos) — overlay product hotspots on playback
- [Playlists & recommendations](https://imagekit.io/docs/video-player/playlist-and-recommendations)
- Floating/sticky player and responsive layouts

Ask `search_docs` things like *"embed ImageKit video player in React"*, *"auto-generate chapters in the video player"*, or *"add shoppable product hotspots to a video"*.

### Embeddable Widgets

- **[Embeddable Media Library widget](https://imagekit.io/docs/dam/embeddable-media-library-widget)** — drop ImageKit's media library picker into your own app so users browse and select assets. Query e.g. *"embed media library widget and handle the selected asset callback"*.
- **[Upload widget](https://github.com/imagekit-samples/uppy-uploader)** — [Uppy](https://uppy.io/)-based widget to upload from local files, URLs, Dropbox, Google Drive, Instagram, and more straight to your media library.

## AI & Automation

- **[Build with AI](https://imagekit.io/docs/build-with-ai)** — agent skills + two MCP servers (DevTools for docs/transformations, API for managing your media library) for Claude, Cursor, Windsurf, VS Code Copilot, and Codex.
- **[n8n](https://imagekit.io/docs/integration/n8n)** — workflow automation.

## URL generation

Beyond the SDKs, ImageKit exposes [URL endpoint functions](https://imagekit.io/docs/url-endpoint-functions) for building and signing delivery URLs directly. Useful when you only need URL construction without pulling in a full SDK. Query e.g. *"sign a delivery URL with an expiry"* or *"build a transformation URL with overlays"*.

---

Query `mcp_imagekit_devtools_search_docs` for implementation details on any of the above integrations, players, widgets, and URL functions. For example: *"generate signed URLs in Node.js"*, *"responsive srcset in React"*, or *"embed the video player with auto subtitles"*.
