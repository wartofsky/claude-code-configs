# Claude Code Configurations

Modular Claude Code configurations with specialized agents, skills, and commands for different tech stacks.

These stacks are designed to be copied into application repos as a starting `.claude/` configuration. Project-level instructions should be written individually in each target project, not shared from this repo.

## Quick Start

```bash
# Copy a stack to your project
cp -r stacks/react-vite-dashboard/.claude /path/to/my-project/
```

Then create or update the target project's own `CLAUDE.md` / `AGENTS.md` with project-specific conventions, scripts, environment variables, and architecture notes.

## Available Stacks

| Stack | Best For | Key Technologies |
|---|---|---|
| `react-tailwind` | General React UI/SPAs | React 19, TypeScript, Tailwind CSS v4, Vitest |
| `react-vite-dashboard` | Authenticated admin/ops dashboards | React 19, Vite 7, Tailwind v4, TanStack Query v5, React Router v7, Okta/OIDC |
| `nextjs-fullstack` | Product apps with app-owned DB/auth | Next.js App Router, React 19, Prisma, Auth.js, Vitest, Playwright |
| `nextjs-cms-frontend` | Public CMS-driven websites | Next.js App Router, external Payload/CMS, Tailwind v4, i18n, SEO, cache tags, Playwright |
| `payload-cms` | Production Payload CMS apps | Payload CMS 3, Next.js, PostgreSQL/MongoDB, Local API, S3/CDN, Lexical, migrations |
| `python-fastapi` | General REST API backend | FastAPI, Pydantic v2, SQLAlchemy 2.x, Alembic, pytest |
| `python-scraping` | AI scraping/data pipeline APIs | FastAPI, OpenAI structured extraction, scraping, SSRF safety, task managers, Salesforce, metrics |
| `python-scripts` | Automation scripts and CLIs | Python 3.12+, Click/Typer, Rich, httpx, pydantic-settings |
| `nestjs-backend` | Enterprise NestJS APIs | NestJS 11, TypeORM, PostgreSQL 16, Swagger, Jest, Docker, Okta/JWT patterns |

## Stack Selection Guide

| If your project is... | Use |
|---|---|
| A React/Vite dashboard with login, filters, polling, tables, and external API calls | `react-vite-dashboard` |
| A simple/component-focused React + Tailwind frontend | `react-tailwind` |
| A public marketing/content site fed by Payload or another CMS | `nextjs-cms-frontend` |
| A full-stack Next.js app with app-owned database/auth/business logic | `nextjs-fullstack` |
| A Payload admin/content backend | `payload-cms` |
| A normal FastAPI CRUD/API service | `python-fastapi` |
| A FastAPI service doing scraping, AI extraction, imports/exports, background jobs, or Salesforce sync | `python-scraping` |
| A NestJS/PostgreSQL API | `nestjs-backend` |

## Structure

Each stack is self-contained:

```text
stacks/<stack>/
└── .claude/
    ├── agents/                # Specialized sub-agents for the stack
    ├── skills/                # Reusable framework/domain guidance
    └── commands/              # Slash commands such as /review-changes
```

## Agent Categories

### Shared Agents

Most stacks include variants of:

- `feature-developer` — implement complete features from specs.
- `code-reviewer` — read-only review before commits/PRs.
- `test-writer` — unit/integration/E2E tests.
- `security-reviewer` — stack-specific security audit.
- `performance-optimizer` — stack-specific performance analysis.
- `refactor-agent` — behavior-preserving cleanup.
- `docs-writer` — README, API, architecture, and runbook docs.
- `git-commit` — Conventional Commit messages.

### Stack-Specific Highlights

| Stack | Specialized Agents |
|---|---|
| `react-vite-dashboard` | `api-client-agent`, `tanstack-query-agent`, `react-router-agent`, `okta-auth-agent`, `dashboard-ux-agent`, `tailwind-reviewer`, `accessibility-reviewer` |
| `nextjs-cms-frontend` | `cms-integration-agent`, `block-renderer-agent`, `i18n-seo-agent`, `api-developer`, `tailwind-reviewer`, `accessibility-reviewer` |
| `payload-cms` | `content-modeler`, `access-control-reviewer`, `migration-types-agent`, `storage-cdn-agent`, `translation-agent`, `local-api-reviewer`, `api-developer` |
| `python-scraping` | `scraper-dev`, `task-manager-dev`, `openai-extraction-agent`, `ssrf-security-reviewer`, `xlsx-import-export-agent`, `salesforce-agent`, `okta-auth-agent`, `ops-monitoring-agent` |
| `nestjs-backend` | `config-validator`, `db-schema-designer`, `migration-runner`, `docker-setup`, `nestjs-auth-setup`, `nestjs-module-generator`, `crud-generator`, `api-developer` |
| `python-fastapi` | `config-validator`, `migration-runner`, `openapi-contract-reviewer` |
| `react-tailwind` | `component-builder`, `tailwind-reviewer`, `tanstack-query-agent`, `react-router-agent` |
| `python-scripts` | `script-developer` |

## Skills by Stack

Each stack includes local custom skills plus curated external skills downloaded from `skills.sh`. See [`docs/skills-matrix.md`](docs/skills-matrix.md) for source links, rationale, and priority.

| Stack | Core Skill Themes |
|---|---|
| `react-tailwind` | React 19, Tailwind v4/design systems, TypeScript, composition, UI/a11y review |
| `react-vite-dashboard` | React/Vite dashboards, TanStack Query, React Router v7, Okta/OIDC, Tailwind v4, Playwright, a11y |
| `nextjs-fullstack` | Next.js, Prisma, React/Next best practices, TypeScript, Node backend patterns, Postgres, migrations, Playwright |
| `nextjs-cms-frontend` | CMS-driven Next.js, React/Next best practices, block rendering, Tailwind v4, TypeScript, Playwright, a11y |
| `payload-cms` | Payload CMS, TypeScript, Node backend patterns, Postgres, migrations, React/admin UI, Playwright |
| `python-fastapi` | Official FastAPI, SQLAlchemy, uv, pytest, Postgres, migrations |
| `python-scraping` | FastAPI, custom scraping/OpenAI extraction/task guidance, uv, pytest, Postgres, migrations |
| `python-scripts` | CLI automation, uv, pytest |
| `nestjs-backend` | NestJS, TypeORM, Node backend patterns, TypeScript, Postgres, migrations |

## Commands

Common commands:

- `/review-changes` — review git changes before commit/PR.
- `/new-component` — create a React component for React stacks.
- `/new-block` — create/register a CMS block component in `nextjs-cms-frontend`.

## Usage Examples

### React Vite Dashboard

```text
"Add a task monitor page with polling and filters"
→ tanstack-query-agent + feature-developer build query hooks, URL state, route, UI states, and tests.

"Implement Okta protected admin routes"
→ okta-auth-agent + react-router-agent wire callback, RequireAuth, role gates, and API headers.
```

### Next.js CMS Frontend

```text
"Add a hero block from Payload CMS"
→ block-renderer-agent creates the component, transform, registry entry, fallback, and tests.

"Fix hreflang/canonical for Canada French pages"
→ i18n-seo-agent audits route builders and metadata generation.
```

### Payload CMS

```text
"Add a localized Blog collection with editor roles"
→ content-modeler + access-control-reviewer define fields, access rules, migrations, and tests.

"Review Local API calls before deploy"
→ local-api-reviewer checks `overrideAccess` behavior, depth, pagination, and draft access.
```

### Python AI Scraping Pipeline

```text
"Add a background XLSX import task"
→ xlsx-import-export-agent + task-manager-dev implement chunking, checkpoints, summaries, and tests.

"Harden school website scraping"
→ scraper-dev + ssrf-security-reviewer add URL validation, rate limits, fixtures, and security tests.
```

### NestJS Backend

```text
"Add a book export module"
→ nestjs-module-generator + db-schema-designer + test-writer create module, DTOs, service, migrations, and tests.

"Prepare backend for production deploy"
→ config-validator + docker-setup review env validation, health checks, Dockerfile, CORS, and Swagger exposure.
```

## Customization

After copying a stack:

1. Create/update the target project's own `CLAUDE.md` or `AGENTS.md` with project-specific architecture, commands, and constraints.
2. Remove agents that do not apply to the project.
3. Add domain-specific agents only when a pattern will repeat.
4. Keep secrets out of `.claude/` files; use placeholders and `.env.example` docs.

## Notes on Current Best Practices

- Tailwind CSS v4 prefers CSS-first configuration with `@theme`; Vite projects should use `@tailwindcss/vite`.
- TanStack Query v5 should use query-key factories, precise invalidation, explicit pending/error/empty states, and controlled polling.
- React Router v7 supports declarative/data/framework modes; dashboards should use nested layouts, lazy route loading, protected routes, and URL search params for shareable state.
- Payload Local API calls that pass a `user` must set `overrideAccess: false` when access control should be enforced.
- Next.js CMS frontends should default to Server Components and centralize CMS fetching, cache tags, and SEO helpers.
- Scraping/data-pipeline APIs must treat URLs, uploaded files, scraped HTML, and AI inputs as untrusted.

## License

MIT
