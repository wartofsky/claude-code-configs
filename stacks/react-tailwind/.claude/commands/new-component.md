# Create New Component

Create a new React 19 component with the following structure:

## Requirements from user
$ARGUMENTS

## Instructions

1. **Determine component type:**
   - UI component → `src/components/ui/`
   - Feature component → `src/components/[feature]/` or `src/features/[feature]/components/`
   - Page component → `src/app/`

2. **Create the component file** with this structure:
   ```tsx
   interface [Name]Props {
     // Props with JSDoc comments
   }

   export function [Name]({ ...props }: [Name]Props) {
     // Implementation
   }
   ```

3. **Apply these standards:**
   - TypeScript strict mode
   - Tailwind for styling
   - Accessible (ARIA, keyboard nav)
   - Responsive (mobile-first)

4. **Include:**
   - Clear prop types
   - Default values where sensible
   - Loading/error states if async
   - Hover/focus/active states

5. **If interactive:**
   - Add `'use client'` directive
   - Use React 19 hooks appropriately

6. **Create a test file** alongside: `[Name].test.tsx`

## Output

Create the component and confirm:
- File path
- Props interface
- Key features implemented
