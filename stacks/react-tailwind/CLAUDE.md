# Project Context

> **Note:** This stack is for React SPA projects (Vite, CRA) or can be used with Next.js. For full-stack Next.js with database and auth, use the `nextjs-fullstack` stack instead.

## Tech Stack

- **Framework**: React 19 (Vite or Next.js)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS
- **Package Manager**: npm / pnpm
- **Testing**: Vitest + Testing Library

## Project Structure

```
src/
├── main.tsx                # Vite entry point (if using Vite)
├── App.tsx                 # Root component (Vite)
├── app/                    # Next.js App Router (if using Next.js)
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── components/
│   ├── ui/                 # Reusable UI components
│   └── [feature]/          # Feature-specific components
├── features/               # Feature modules
│   └── [feature-name]/
│       ├── components/
│       ├── hooks/
│       ├── types.ts
│       └── api.ts
├── hooks/                  # Shared custom hooks
├── lib/                    # Utilities and helpers
├── types/                  # Global TypeScript types
└── test/                   # Test utilities and mocks
```

## Code Conventions

### File Naming
- Components: `PascalCase.tsx`
- Hooks: `use[Name].ts`
- Utilities: `camelCase.ts`
- Types: `types.ts` or `[name].types.ts`

### Component Structure
```tsx
// 1. Imports (external, then internal)
// 2. Types/Interfaces
// 3. Component function
// 4. Hooks at top
// 5. Handlers
// 6. Render
```

### TypeScript Rules
- No `any` - use `unknown` and narrow
- Explicit return types on exported functions
- Interfaces for component props
- `type` for unions and intersections

## Commands

```bash
# Development
npm run dev

# Build
npm run build

# Test
npm run test
npm run test:watch
npm run test:coverage

# Lint
npm run lint
npm run lint:fix

# Type check
npm run typecheck
```

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_API_URL=
DATABASE_URL=
```

## Common Patterns

### Data Fetching
- Server Components for initial data
- TanStack Query for client-side caching
- Server Actions for mutations

### Forms
- React 19 `useActionState` + `useFormStatus`
- Zod for validation
- Server Actions for submission

### State Management
- React Context for simple global state
- Zustand for complex client state
- URL state for filters/pagination

## Notes

<!-- Add project-specific notes here -->
