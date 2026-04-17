---
name: nextjs-cms-frontend
description: >
  Next.js App Router patterns for CMS-driven public websites.
  Trigger: Next.js CMS pages, external Payload/CMS fetching, block renderers, i18n, SEO, cache tags, ISR, preview, or revalidation.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Critical Patterns
- Default to Server Components; add client boundaries only for interactivity.
- Centralize CMS fetches and normalize raw payloads into UI DTOs.
- Use typed block registries and safe unknown-block fallbacks.
- Choose explicit caching: static, ISR, dynamic, or no-store.
- Secure preview/revalidation endpoints with secrets/signatures.
- Generate canonical, hreflang, metadata, sitemap, and structured data server-side.

## Commands
```bash
npm run lint
npm run type-check
npm run test:run
npm run build
npm run test:e2e
```
