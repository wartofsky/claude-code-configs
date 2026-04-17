# Stack Skills Matrix

Last reviewed: 2026-04-17

This matrix documents the curated third-party skills copied into each stack from `skills.sh`, plus local/custom stack skills that remain necessary because they encode project-specific patterns.

## Selection Policy

Skills are installed only when they meet at least one of these criteria:

- High-relevance framework best practices from a trusted/active source.
- Cross-stack value that complements stack-specific agents.
- Covers a recurring workflow that should not be reinvented per project.
- Does not impose incompatible assumptions on the stack.

Rejected/deferred candidates are listed at the end so we do not re-evaluate them repeatedly.

## Curated Sources

| Skill | Source | skills.sh | Why selected |
|---|---|---|---|
| `vercel-react-best-practices` | `vercel-labs/agent-skills` | https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices | React/Next performance and rendering best practices from Vercel. |
| `vercel-composition-patterns` | `vercel-labs/agent-skills` | https://skills.sh/vercel-labs/agent-skills/vercel-composition-patterns | React composition patterns, including React 19 notes. |
| `web-design-guidelines` | `vercel-labs/agent-skills` | https://skills.sh/vercel-labs/agent-skills/web-design-guidelines | UI/UX/accessibility review guidance. |
| `playwright-best-practices` | `currents-dev/playwright-best-practices-skill` | https://skills.sh/currents-dev/playwright-best-practices-skill/playwright-best-practices | High-install E2E testing best practices. |
| `tailwind-design-system` | `wshobson/agents` | https://skills.sh/wshobson/agents/tailwind-design-system | Tailwind CSS v4 design-system guidance. |
| `typescript-advanced-types` | `wshobson/agents` | https://skills.sh/wshobson/agents/typescript-advanced-types | Advanced TypeScript patterns for strongly typed apps. |
| `nodejs-backend-patterns` | `wshobson/agents` | https://skills.sh/wshobson/agents/nodejs-backend-patterns | Production Node backend patterns for API stacks. |
| `uv-package-manager` | `wshobson/agents` | https://skills.sh/wshobson/agents/uv-package-manager | Modern Python dependency and virtualenv management. |
| `fastapi` | `fastapi/fastapi` | https://skills.sh/fastapi/fastapi/fastapi | Official FastAPI skill with current framework guidance. |
| `python-testing` | `affaan-m/everything-claude-code` | https://skills.sh/affaan-m/everything-claude-code/python-testing | Pytest/TDD/fixtures/mocking/coverage guidance. |
| `supabase-postgres-best-practices` | `supabase/agent-skills` | https://skills.sh/supabase/agent-skills/supabase-postgres-best-practices | Postgres performance and schema best practices. Useful beyond Supabase. |
| `database-migrations-sql-migrations` | `sickn33/antigravity-awesome-skills` | https://skills.sh/sickn33/antigravity-awesome-skills/database-migrations-sql-migrations | Zero-downtime SQL migration strategy. |
| `nestjs-best-practices` | `kadajett/agent-nestjs-skills` | https://skills.sh/kadajett/agent-nestjs-skills/nestjs-best-practices | NestJS architecture and production rules. |
| `accessibility-auditor` | `patricio0312rev/skills` | https://skills.sh/patricio0312rev/skills/accessibility-auditor | Detailed WCAG/ARIA/keyboard accessibility patterns. |

## Stack Assignments

### `react-tailwind`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `react19` | local | Existing React 19 stack guidance. |
| core | `tailwind` | local | Existing Tailwind basics. |
| core | `vercel-react-best-practices` | external | React performance/rendering guidance. |
| core | `tailwind-design-system` | external | Tailwind v4 design-system guidance. |
| recommended | `vercel-composition-patterns` | external | Scalable component composition. |
| recommended | `typescript-advanced-types` | external | Strong typing for components/hooks. |
| recommended | `web-design-guidelines` | external | UI/UX review. |
| recommended | `accessibility-auditor` | external | Deeper a11y audits. |
| optional | `tanstack-query` | local | Useful when SPA uses server state. |

### `react-vite-dashboard`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `react-vite-dashboard` | local | Stack-specific dashboard architecture. |
| core | `tanstack-query` | local | Query keys, polling, invalidation. |
| core | `react-router-v7` | local | Route layouts, protected routes, URL state. |
| core | `okta-spa` | local | Okta/OIDC SPA specifics. |
| core | `tailwind-v4` | local | CSS-first Tailwind v4 guidance. |
| core | `vercel-react-best-practices` | external | React performance. |
| core | `tailwind-design-system` | external | Dashboard design-system patterns. |
| core | `typescript-advanced-types` | external | Typed API/hooks/routes. |
| recommended | `playwright-best-practices` | external | E2E dashboard workflows. |
| recommended | `vercel-composition-patterns` | external | Reusable UI APIs. |
| recommended | `web-design-guidelines` | external | UI/UX reviews. |
| recommended | `accessibility-auditor` | external | Forms/tables/modals a11y. |

### `nextjs-cms-frontend`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `nextjs-cms-frontend` | local | External CMS/block/i18n/SEO stack specifics. |
| core | `vercel-react-best-practices` | external | Next/React rendering and performance. |
| core | `vercel-composition-patterns` | external | Component composition at site scale. |
| core | `tailwind-v4` | local | CSS-first Tailwind v4 guidance. |
| core | `tailwind-design-system` | external | Design-system tokens/components. |
| core | `playwright-best-practices` | external | E2E for public journeys. |
| recommended | `typescript-advanced-types` | external | CMS DTOs/block unions. |
| recommended | `web-design-guidelines` | external | Public site UX/a11y review. |
| recommended | `accessibility-auditor` | external | WCAG/ARIA/keyboard audits. |
| optional | `playwright-vitest` | local | Local testing quick-reference retained. |

### `nextjs-fullstack`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `nextjs` | local | Existing Next.js App Router stack guidance. |
| core | `prisma` | local | Existing Prisma stack guidance. |
| core | `vercel-react-best-practices` | external | React/Next performance. |
| core | `typescript-advanced-types` | external | App/domain typing. |
| core | `nodejs-backend-patterns` | external | Route handlers/API backend patterns. |
| core | `supabase-postgres-best-practices` | external | Postgres schema/query guidance. |
| core | `database-migrations-sql-migrations` | external | Migration safety. |
| recommended | `playwright-best-practices` | external | E2E coverage. |
| recommended | `tailwind-design-system` | external | UI/design systems. |
| recommended | `vercel-composition-patterns` | external | Component composition. |
| recommended | `web-design-guidelines` | external | UX/a11y review. |
| recommended | `accessibility-auditor` | external | Detailed accessibility audits. |

### `payload-cms`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `payload` | local | Existing Payload CMS guidance. |
| core | `typescript-advanced-types` | external | Collection/block/admin typing. |
| core | `nodejs-backend-patterns` | external | Custom route handlers/services. |
| core | `supabase-postgres-best-practices` | external | Postgres adapter performance/schema guidance. |
| core | `database-migrations-sql-migrations` | external | Migration safety. |
| recommended | `vercel-react-best-practices` | external | Payload-on-Next React/admin UI patterns. |
| recommended | `vercel-composition-patterns` | external | Admin/custom components. |
| recommended | `playwright-best-practices` | external | Admin/editor E2E workflows. |
| recommended | `web-design-guidelines` | external | Admin UX review. |
| recommended | `accessibility-auditor` | external | Admin/editor a11y. |

### `nestjs-backend`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `nestjs` | local | Existing NestJS stack guidance. |
| core | `typeorm` | local | Existing TypeORM guidance. |
| core | `nestjs-best-practices` | external | Production NestJS architecture and rules. |
| core | `nodejs-backend-patterns` | external | Node backend operations/security. |
| core | `typescript-advanced-types` | external | DTOs/types/utilities. |
| core | `supabase-postgres-best-practices` | external | Postgres guidance. |
| core | `database-migrations-sql-migrations` | external | Migration safety. |

### `python-fastapi`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `fastapi` | external | Official FastAPI best-practices skill. |
| core | `sqlalchemy` | local | Existing SQLAlchemy guidance. |
| core | `python-testing` | external | Pytest/fixtures/mocking/coverage. |
| core | `uv-package-manager` | external | Modern dependency management. |
| core | `supabase-postgres-best-practices` | external | Postgres guidance. |
| core | `database-migrations-sql-migrations` | external | Migration safety. |
| optional | `fastapi-python` | external/pre-existing | Extra FastAPI overview; less specific. |
| optional | `fastapi-templates` | external/pre-existing | Useful for greenfield projects; assumes async patterns, so use carefully. |

### `python-scraping`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `fastapi` | external | Official FastAPI guidance. |
| core | `web-scraping` | local | Existing custom httpx/BeautifulSoup scraping patterns retained over weaker marketplace version. |
| core | `openai-extraction` | local | Existing custom Responses/structured-output/Pydantic extraction guidance. |
| core | `background-tasks` | local | Existing task orchestration guidance. |
| core | `python-testing` | external | Pytest/mocking/coverage. |
| core | `uv-package-manager` | external | Python dependency management. |
| core | `supabase-postgres-best-practices` | external | Postgres performance/schema guidance. |
| core | `database-migrations-sql-migrations` | external | Migration safety. |
| optional | `websockets` | local | Keep only for projects that actually use WebSockets. |

### `python-scripts`

| Priority | Skill | Type | Reason |
|---|---|---|---|
| core | `cli` | local | Existing CLI/automation guidance. |
| core | `uv-package-manager` | external | Dependency/venv management. |
| core | `python-testing` | external | Pytest/fixtures/mocking. |

## Deferred / Rejected Candidates

| Candidate | Decision | Reason |
|---|---|---|
| `structured-output-extractor` | deferred | Marketplace skill is mostly Chat Completions/Zod/function-calling oriented; our custom `openai-extraction` is more current for Responses API + Pydantic. |
| `mindrally/skills@web-scraping` | rejected for install | Too shallow compared with existing custom `python-scraping/.claude/skills/web-scraping`; staging copy was reviewed but not installed. |
| `gemini/next-best-practices` | rejected | CLI clone failed due GitHub auth/access issue; existing local `nextjs` plus Vercel React skills cover current needs. |
| `fastapi-templates` on `python-scraping` | deferred | Useful but assumes async greenfield FastAPI; the scraping stack must support sync or async SQLAlchemy based on project reality. |

## Maintenance

- Re-run `npx skills find <topic>` before adding new external skills.
- Install first into `/tmp/skills-staging` and inspect `SKILL.md` before copying into stacks.
- Prefer replacing stale marketplace skills with local custom skills when the project requires newer APIs or stricter patterns.
- Review this matrix every 30-60 days or before major framework upgrades.
