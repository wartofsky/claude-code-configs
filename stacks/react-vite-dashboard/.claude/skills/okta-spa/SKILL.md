---
name: okta-spa
description: >
  Okta/OIDC SPA authentication patterns.
  Trigger: Okta, OIDC, login, logout, callback, token, auth header, protected route, role gate, or JWKS.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## Critical Patterns
- Use authorization code + PKCE through the provider SDK.
- Never store tokens manually outside the SDK/token manager.
- Centralize token attachment in the API client.
- Gate admin UI from backend-confirmed roles.
- Handle callback, renewal, logout, 401, and 403 flows explicitly.
