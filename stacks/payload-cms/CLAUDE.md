# Payload CMS Project

Headless CMS built with Payload 3.0 and Next.js.

## Tech Stack

- **CMS**: Payload 3.0
- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript
- **Database**: MongoDB / PostgreSQL
- **Editor**: Lexical (Rich Text)
- **Storage**: S3 / Local

## Project Structure

```
src/
├── app/
│   ├── (frontend)/          # Public frontend
│   │   ├── page.tsx
│   │   └── posts/
│   │       └── [slug]/
│   └── (payload)/           # Admin panel
│       └── admin/
│           └── [[...segments]]/
├── collections/
│   ├── Users.ts
│   ├── Posts.ts
│   ├── Media.ts
│   ├── Categories.ts
│   └── index.ts
├── globals/
│   ├── Settings.ts
│   └── Navigation.ts
├── blocks/
│   ├── Hero.ts
│   ├── Content.ts
│   └── CTA.ts
├── fields/
│   ├── slug.ts
│   └── meta.ts
├── hooks/
│   ├── beforeChange/
│   └── afterChange/
├── access/
│   └── index.ts
├── components/
│   └── admin/
├── payload.config.ts
└── payload-types.ts         # Generated
```

## Conventions

### Collections
- PascalCase for collection files
- Plural names for slug (`posts`, `users`)
- `useAsTitle` for admin display
- Group related collections

### Fields
- Reusable fields in `/fields`
- Use `admin.position: 'sidebar'` for metadata
- Add `description` for complex fields

### Access Control
- Default to restrictive
- Check `role` for admin features
- Use field-level access for sensitive data

### Hooks
- `beforeChange`: data modification, auto-fill
- `afterChange`: cache revalidation, webhooks

## Commands

```bash
# Development
pnpm dev

# Generate types
pnpm payload generate:types

# Database
pnpm payload migrate:create
pnpm payload migrate

# Build
pnpm build
```

## Environment

```bash
# .env
DATABASE_URI=mongodb://localhost/payload
PAYLOAD_SECRET=your-secret-key

# S3 (optional)
S3_BUCKET=
S3_ACCESS_KEY=
S3_SECRET_KEY=
S3_REGION=
```

## Local API Usage

```typescript
import { getPayload } from 'payload'
import config from '@/payload.config'

// In Server Component
const payload = await getPayload({ config })

const { docs } = await payload.find({
  collection: 'posts',
  where: { status: { equals: 'published' } },
})
```

## Patterns

### Revalidation
```typescript
// In afterChange hook
revalidatePath(`/posts/${doc.slug}`)
revalidateTag('posts')
```

### Draft Preview
```typescript
// Query with draft: true
const preview = await payload.findByID({
  collection: 'posts',
  id,
  draft: true,
})
```

## Notes

<!-- Project-specific notes -->
