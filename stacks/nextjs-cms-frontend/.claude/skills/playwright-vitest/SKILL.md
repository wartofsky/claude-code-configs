---
name: playwright-vitest
description: >
  Testing patterns for React/Next.js with Vitest, Testing Library, and Playwright.
  Trigger: tests, E2E, unit tests, integration tests, mocks, fixtures, or accessibility checks.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Critical Patterns
- Unit test pure transforms, helpers, hooks, and client components.
- E2E test user-visible journeys; mock third parties at boundaries only.
- Keep fixtures small and close to real API/CMS shapes.
- Assert behavior and accessible UI, not implementation details.
