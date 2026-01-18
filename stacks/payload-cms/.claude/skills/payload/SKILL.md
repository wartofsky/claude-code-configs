---
name: payload
description: Payload CMS 3.0 patterns for collections, fields, hooks, and access control. Use when building CMS features, content models, or admin customization. Triggers on Payload, collection, CMS, content model keywords.
---

# Payload CMS 3.0 Patterns

## Quick Reference

```typescript
import type { 
  CollectionConfig, 
  GlobalConfig,
  Field,
  Block,
  Access,
  CollectionBeforeChangeHook,
  CollectionAfterChangeHook,
} from 'payload'
```

## Field Types

```typescript
// Text
{ name: 'title', type: 'text', required: true, maxLength: 100 }

// Textarea
{ name: 'description', type: 'textarea' }

// Rich Text (Lexical)
{ name: 'content', type: 'richText' }

// Number
{ name: 'price', type: 'number', min: 0 }

// Select
{
  name: 'status',
  type: 'select',
  options: [
    { label: 'Draft', value: 'draft' },
    { label: 'Published', value: 'published' },
  ],
  defaultValue: 'draft',
}

// Checkbox
{ name: 'featured', type: 'checkbox', defaultValue: false }

// Date
{
  name: 'publishedAt',
  type: 'date',
  admin: {
    date: { pickerAppearance: 'dayAndTime' }
  }
}

// Upload (Image/File)
{ name: 'image', type: 'upload', relationTo: 'media' }

// Relationship
{
  name: 'author',
  type: 'relationship',
  relationTo: 'users',
  hasMany: false,
}

// Relationships (multiple)
{
  name: 'categories',
  type: 'relationship',
  relationTo: 'categories',
  hasMany: true,
}

// Array
{
  name: 'links',
  type: 'array',
  fields: [
    { name: 'label', type: 'text' },
    { name: 'url', type: 'text' },
  ],
}

// Group
{
  name: 'meta',
  type: 'group',
  fields: [
    { name: 'title', type: 'text' },
    { name: 'description', type: 'textarea' },
  ],
}

// Blocks (flexible content)
{
  name: 'layout',
  type: 'blocks',
  blocks: [HeroBlock, ContentBlock, CTABlock],
}

// JSON
{ name: 'metadata', type: 'json' }

// Email
{ name: 'email', type: 'email' }

// Point (geo)
{ name: 'location', type: 'point' }
```

## Admin Configuration

```typescript
const Posts: CollectionConfig = {
  slug: 'posts',
  admin: {
    useAsTitle: 'title',
    defaultColumns: ['title', 'status', 'updatedAt'],
    group: 'Content',  // Group in sidebar
    description: 'Blog posts',
    listSearchableFields: ['title', 'slug'],
    pagination: { defaultLimit: 20 },
    preview: (doc) => `https://example.com/preview/${doc.slug}`,
  },
  // Field-level admin
  fields: [
    {
      name: 'status',
      type: 'select',
      admin: {
        position: 'sidebar',
        description: 'Current publication status',
        condition: (data) => data.title,  // Show only if title exists
      },
    },
  ],
}
```

## Access Control Patterns

```typescript
// Public read
read: () => true

// Logged in only
read: ({ req: { user } }) => Boolean(user)

// Admin only
read: ({ req: { user } }) => user?.role === 'admin'

// Owner or admin
read: ({ req: { user } }) => {
  if (!user) return false
  if (user.role === 'admin') return true
  return { author: { equals: user.id } }
}

// Field-level access
{
  name: 'secretField',
  type: 'text',
  access: {
    read: ({ req: { user } }) => user?.role === 'admin',
    update: ({ req: { user } }) => user?.role === 'admin',
  },
}
```

## Hooks

```typescript
// Before Change - modify data before save
const beforeChange: CollectionBeforeChangeHook = ({
  data,
  req,
  operation,  // 'create' | 'update'
  originalDoc,
}) => {
  if (operation === 'create') {
    data.author = req.user?.id
  }
  return data
}

// After Change - side effects after save
const afterChange: CollectionAfterChangeHook = ({
  doc,
  previousDoc,
  req,
  operation,
}) => {
  if (doc.status === 'published') {
    revalidatePath(`/posts/${doc.slug}`)
  }
  return doc
}

// Before Read - filter/modify queries
const beforeRead: CollectionBeforeReadHook = ({ req, query }) => {
  if (req.user?.role !== 'admin') {
    query.where = {
      ...query.where,
      status: { equals: 'published' },
    }
  }
  return query
}

// After Read - transform data
const afterRead: CollectionAfterReadHook = ({ doc, req }) => {
  if (req.user?.role !== 'admin') {
    delete doc.internalNotes
  }
  return doc
}
```

## Local API

```typescript
import { getPayload } from 'payload'
import config from '@/payload.config'

const payload = await getPayload({ config })

// Find
const { docs, totalDocs } = await payload.find({
  collection: 'posts',
  where: { status: { equals: 'published' } },
  sort: '-createdAt',
  limit: 10,
  page: 1,
  depth: 2,  // Populate relationships
})

// Find by ID
const post = await payload.findByID({
  collection: 'posts',
  id: '123',
})

// Create
const newPost = await payload.create({
  collection: 'posts',
  data: { title: 'New Post', status: 'draft' },
})

// Update
const updated = await payload.update({
  collection: 'posts',
  id: '123',
  data: { status: 'published' },
})

// Delete
await payload.delete({
  collection: 'posts',
  id: '123',
})

// Globals
const settings = await payload.findGlobal({ slug: 'settings' })
await payload.updateGlobal({ slug: 'settings', data: { ... } })
```

## Versions & Drafts

```typescript
const Posts: CollectionConfig = {
  slug: 'posts',
  versions: {
    drafts: {
      autosave: {
        interval: 1500,  // ms
      },
    },
    maxPerDoc: 10,
  },
  // ...
}

// Query draft versions
const draft = await payload.findByID({
  collection: 'posts',
  id: '123',
  draft: true,
})
```

## Lexical Rich Text

```typescript
import { lexicalEditor } from '@payloadcms/richtext-lexical'

// In field
{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      ...defaultFeatures,
      // Add custom features
    ],
  }),
}

// Custom blocks in rich text
import { BlocksFeature } from '@payloadcms/richtext-lexical'

lexicalEditor({
  features: ({ defaultFeatures }) => [
    ...defaultFeatures,
    BlocksFeature({
      blocks: [CallToActionBlock, ImageGalleryBlock],
    }),
  ],
})
```

## TypeScript Generation

```bash
# Generate types
pnpm payload generate:types

# Types are in payload-types.ts
import type { Post, User, Media } from './payload-types'
```
