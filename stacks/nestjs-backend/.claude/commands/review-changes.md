# Review Changes

Review all current git changes before committing or creating a PR.

## Arguments

$ARGUMENTS

## Instructions

1. Get the current diff:
   - If there are staged changes: `git diff --staged`
   - If there are no staged changes: `git diff`
   - If comparing to main: `git diff main...HEAD`

2. Categorize each changed file:
   - **Feature**: New functionality (controllers, services, modules)
   - **Bug Fix**: Error corrections
   - **Refactor**: Code restructuring without behavior change
   - **Test**: New or updated tests
   - **Config**: Configuration changes (env, docker, CI)
   - **Migration**: Database schema changes

3. For each change, verify:

### NestJS Patterns
- [ ] Controllers are thin (logic in services)
- [ ] DTOs have proper class-validator decorators
- [ ] Entities have correct TypeORM decorators and column types
- [ ] Modules properly import/export dependencies
- [ ] Services use constructor injection
- [ ] Guards applied on endpoints that need auth

### TypeScript
- [ ] No `any` types
- [ ] Proper return types on methods
- [ ] Strict null checks handled

### Security
- [ ] No hardcoded secrets or credentials
- [ ] Input validated via DTOs
- [ ] Auth guards on protected endpoints
- [ ] No sensitive data in responses (passwords, tokens)
- [ ] No `console.log` with sensitive data

### Database
- [ ] Migration generated for entity changes
- [ ] Indexes on frequently queried columns
- [ ] Relations loaded explicitly (no lazy loading)
- [ ] Transactions for multi-step operations

### Testing
- [ ] Tests exist for new service methods
- [ ] Edge cases covered (not found, invalid input)
- [ ] Mocks used for external dependencies

### Performance
- [ ] List endpoints paginated
- [ ] No N+1 queries (check relations loading)
- [ ] Select only needed columns for large datasets

## Output Format

```markdown
## Change Review

### Summary
- **Type**: [Feature/Fix/Refactor/etc.]
- **Scope**: [Module(s) affected]
- **Files changed**: [count]

### Changes
[Brief description of each significant change]

### Issues Found

#### Critical
[Must fix before merge]

#### Warnings
[Should fix]

#### Suggestions
[Optional improvements]

### Verdict
[APPROVE / NEEDS CHANGES / BLOCK]
```
