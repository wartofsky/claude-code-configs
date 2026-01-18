---
name: prisma
description: Prisma ORM patterns for database operations in Next.js. Use when creating models, writing queries, handling relations, or managing migrations. Triggers on Prisma, database, model, schema, query, migration keywords.
---

# Prisma ORM Patterns

## Setup

### Client Singleton

```typescript
// lib/db.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const db =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query'] : [],
  })

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = db
}
```

## Schema Design

### Basic Model

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  posts     Post[]
  profile   Profile?

  @@index([email])
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  author    User     @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  String

  categories Category[]

  @@index([authorId])
  @@index([published])
}

model Profile {
  id     String  @id @default(cuid())
  bio    String?
  avatar String?

  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  userId String @unique
}

model Category {
  id    String @id @default(cuid())
  name  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
}
```

## CRUD Operations

### Create

```typescript
// Single create
const user = await db.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
  },
})

// Create with relation
const userWithProfile = await db.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    profile: {
      create: {
        bio: 'Hello world',
      },
    },
  },
  include: {
    profile: true,
  },
})

// Create many
const users = await db.user.createMany({
  data: [
    { email: 'user1@example.com' },
    { email: 'user2@example.com' },
  ],
  skipDuplicates: true,
})
```

### Read

```typescript
// Find unique
const user = await db.user.findUnique({
  where: { email: 'user@example.com' },
})

// Find unique or throw
const user = await db.user.findUniqueOrThrow({
  where: { id: userId },
})

// Find first
const user = await db.user.findFirst({
  where: { role: 'ADMIN' },
})

// Find many with filters
const posts = await db.post.findMany({
  where: {
    published: true,
    author: {
      email: {
        contains: '@example.com',
      },
    },
  },
  orderBy: {
    createdAt: 'desc',
  },
  take: 10,
  skip: 0,
})

// With relations
const userWithPosts = await db.user.findUnique({
  where: { id: userId },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
    },
    profile: true,
  },
})

// Select specific fields
const userEmails = await db.user.findMany({
  select: {
    id: true,
    email: true,
  },
})
```

### Update

```typescript
// Update one
const user = await db.user.update({
  where: { id: userId },
  data: {
    name: 'Updated Name',
  },
})

// Update many
const result = await db.user.updateMany({
  where: {
    role: 'USER',
  },
  data: {
    verified: true,
  },
})

// Upsert (update or create)
const user = await db.user.upsert({
  where: { email: 'user@example.com' },
  update: { name: 'Updated' },
  create: { email: 'user@example.com', name: 'New User' },
})

// Update relations
const post = await db.post.update({
  where: { id: postId },
  data: {
    categories: {
      connect: [{ id: categoryId }],
      // or disconnect, set, create
    },
  },
})
```

### Delete

```typescript
// Delete one
const user = await db.user.delete({
  where: { id: userId },
})

// Delete many
const result = await db.user.deleteMany({
  where: {
    createdAt: {
      lt: new Date('2023-01-01'),
    },
  },
})
```

## Advanced Queries

### Filtering

```typescript
const posts = await db.post.findMany({
  where: {
    // AND (implicit)
    published: true,
    title: { contains: 'prisma', mode: 'insensitive' },

    // OR
    OR: [
      { title: { contains: 'database' } },
      { content: { contains: 'database' } },
    ],

    // NOT
    NOT: { authorId: excludedUserId },

    // Relations
    author: {
      email: { endsWith: '@company.com' },
    },

    // Array contains
    categories: {
      some: { name: 'Technology' },
    },
  },
})
```

### Aggregations

```typescript
// Count
const userCount = await db.user.count({
  where: { role: 'USER' },
})

// Aggregate
const stats = await db.post.aggregate({
  _count: { id: true },
  _avg: { views: true },
  _max: { views: true },
  where: { published: true },
})

// Group by
const postsByAuthor = await db.post.groupBy({
  by: ['authorId'],
  _count: { id: true },
  _sum: { views: true },
  orderBy: {
    _count: { id: 'desc' },
  },
  having: {
    id: { _count: { gt: 5 } },
  },
})
```

### Pagination

```typescript
// Offset pagination
const posts = await db.post.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: { createdAt: 'desc' },
})

// Cursor pagination (more efficient for large datasets)
const posts = await db.post.findMany({
  take: pageSize,
  skip: 1, // Skip the cursor
  cursor: { id: lastPostId },
  orderBy: { createdAt: 'desc' },
})

// With total count
const [posts, total] = await db.$transaction([
  db.post.findMany({
    skip: (page - 1) * pageSize,
    take: pageSize,
  }),
  db.post.count(),
])
```

## Transactions

```typescript
// Sequential transactions
const [user, post] = await db.$transaction([
  db.user.create({ data: { email: 'new@example.com' } }),
  db.post.create({ data: { title: 'Hello', authorId: 'temp' } }),
])

// Interactive transactions
const result = await db.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'new@example.com' },
  })

  const post = await tx.post.create({
    data: {
      title: 'Hello',
      authorId: user.id,
    },
  })

  // If anything throws, entire transaction rolls back
  if (!post) {
    throw new Error('Failed to create post')
  }

  return { user, post }
})
```

## Raw Queries

```typescript
// Raw query
const users = await db.$queryRaw<User[]>`
  SELECT * FROM "User" WHERE email LIKE ${`%@${domain}`}
`

// Raw execute
const result = await db.$executeRaw`
  UPDATE "User" SET "lastLogin" = NOW() WHERE id = ${userId}
`

// With Prisma.sql for dynamic queries
import { Prisma } from '@prisma/client'

const orderBy = Prisma.sql`ORDER BY "createdAt" DESC`
const users = await db.$queryRaw`SELECT * FROM "User" ${orderBy}`
```

## Server Actions Pattern

```typescript
// actions/posts.ts
'use server'

import { db } from '@/lib/db'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().optional(),
})

export async function createPost(formData: FormData) {
  const validated = CreatePostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  })

  if (!validated.success) {
    return { error: validated.error.flatten().fieldErrors }
  }

  const post = await db.post.create({
    data: {
      ...validated.data,
      authorId: await getCurrentUserId(),
    },
  })

  revalidatePath('/posts')
  return { data: post }
}
```

## Server Component Pattern

```typescript
// app/posts/page.tsx
import { db } from '@/lib/db'

export default async function PostsPage() {
  const posts = await db.post.findMany({
    where: { published: true },
    include: { author: { select: { name: true } } },
    orderBy: { createdAt: 'desc' },
    take: 10,
  })

  return (
    <div>
      {posts.map((post) => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>By {post.author.name}</p>
        </article>
      ))}
    </div>
  )
}
```

## Migrations

```bash
# Create migration
npx prisma migrate dev --name add_user_role

# Apply migrations (production)
npx prisma migrate deploy

# Reset database (development only!)
npx prisma migrate reset

# Generate client
npx prisma generate

# Open Prisma Studio
npx prisma studio

# Format schema
npx prisma format
```

## Best Practices

1. **Use transactions** for multiple related operations
2. **Select only needed fields** to reduce data transfer
3. **Use indexes** for frequently queried fields
4. **Cursor pagination** for large datasets
5. **Soft deletes** for recoverable data (add `deletedAt` field)
6. **Timestamps** with `@default(now())` and `@updatedAt`
7. **Cascade deletes** carefully - use `onDelete: Cascade` intentionally
8. **Type-safe queries** - let TypeScript catch errors

## Common Patterns

### Soft Delete

```prisma
model Post {
  id        String    @id @default(cuid())
  deletedAt DateTime?
  // ...
}
```

```typescript
// "Delete"
await db.post.update({
  where: { id },
  data: { deletedAt: new Date() },
})

// Query active only
const posts = await db.post.findMany({
  where: { deletedAt: null },
})
```

### Audit Fields

```prisma
model Post {
  id        String   @id @default(cuid())
  createdAt DateTime @default(now())
  createdBy String
  updatedAt DateTime @updatedAt
  updatedBy String?
}
```
