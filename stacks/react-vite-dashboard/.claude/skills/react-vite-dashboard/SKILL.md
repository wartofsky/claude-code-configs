---
name: react-vite-dashboard
description: >
  React 19 + Vite dashboard architecture patterns.
  Trigger: dashboard pages, admin UI, feature modules, forms, tables, filters, task monitors, or SPA architecture.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Critical Patterns
- Centralized providers: QueryClient, Router, Auth, notifications, error boundaries.
- Feature folders own pages/components/services/types/tests.
- Use URL state for shareable dashboard filters and pagination.
- All API calls go through the central API client.
- Build accessible UI primitives for repeated dashboard patterns.
