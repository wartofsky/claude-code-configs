# Review Changes

Review current changes before commit/PR.

## Arguments
$ARGUMENTS

## Instructions

1. **Get the diff**
   ```bash
   git diff --staged  # If staged
   git diff           # If unstaged
   git diff main...HEAD  # Compare to main branch
   ```

2. **Analyze changes by category**
   - New features added
   - Bugs fixed
   - Refactoring done
   - Tests added/modified
   - Config changes

3. **Check for issues**
   - TypeScript/type errors
   - Missing error handling
   - Security concerns
   - Performance issues
   - Missing tests
   - Console.logs or debug code left behind

4. **Provide summary**
   - What changed
   - Potential risks
   - Suggestions for improvement
   - Ready to merge? Yes/No with reasons

## Output Format

```markdown
## Changes Summary

### Files Modified
- `path/to/file.ts` - Brief description

### New Features
- Feature 1

### Bug Fixes
- Fix 1

### ⚠️ Concerns
- Issue 1

### ✅ Ready to Merge
Yes/No - Reason
```
