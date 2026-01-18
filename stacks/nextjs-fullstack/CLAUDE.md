# Project Context

## Tech Stack

- **Framework**: Next.js 15 (App Router)
- **Language**: TypeScript (strict)
- **UI**: React 19, Tailwind CSS
- **Database**: PostgreSQL + Prisma
- **Auth**: NextAuth.js / Auth.js
- **Validation**: Zod
- **Testing**: Vitest, Playwright

## Project Structure

```
src/
├── app/
│   ├── (auth)/              # Auth pages (login, register)
│   ├── (dashboard)/         # Protected app pages
│   ├── api/                  # Route handlers
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── components/
│   ├── ui/                   # Reusable UI (buttons, inputs)
│   └── [feature]/           # Feature components
├── actions/                  # Server Actions
├── lib/
│   ├── db.ts                # Prisma client
│   ├── auth.ts              # Auth config
│   └── utils.ts
├── types/
└── prisma/
    └── schema.prisma
```

## Conventions

### File Naming
- Pages: `page.tsx`
- Layouts: `layout.tsx`
- Components: `PascalCase.tsx`
- Actions: `[feature].ts` in /actions
- API routes: `route.ts`

### Data Flow
1. **Read data**: Server Components with direct DB access
2. **Write data**: Server Actions with validation
3. **External APIs**: Route Handlers
4. **Real-time**: Route Handlers with streaming

### State Management
- Server state: React Server Components
- Form state: useActionState
- URL state: searchParams
- Client state: useState (minimal)

## Commands

```bash
# Development
npm run dev

# Database
npx prisma generate
npx prisma migrate dev
npx prisma studio

# Build
npm run build
npm run start

# Test
npm run test
npm run test:e2e
```

## Environment

```bash
# .env.local
DATABASE_URL=
NEXTAUTH_SECRET=
NEXTAUTH_URL=
```

## Patterns

### Server Actions
- Always validate with Zod
- Always check auth
- Use revalidatePath/revalidateTag after mutations
- Return typed results { success, data?, error? }

### API Routes
- RESTful naming
- Consistent error format
- Proper status codes
- Always try/catch

## Notes

<!-- Project-specific notes -->
