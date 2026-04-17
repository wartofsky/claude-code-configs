---
name: tanstack-query
description: >
  TanStack Query v5 server-state patterns.
  Trigger: useQuery, useMutation, query keys, invalidation, polling, optimistic updates, cache, or server state.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Critical Patterns
- Use query-key factories per feature.
- Use `enabled` for auth/param dependent queries.
- Use `refetchInterval` intentionally for polling; stop on terminal states.
- Invalidate precise query keys after mutations.
- Test hooks with retries disabled and fresh QueryClient instances.
