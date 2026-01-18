---
name: tailwind-modern
description: Modern Tailwind CSS v4 patterns for beautiful, responsive UIs. Use when styling components, creating layouts, implementing animations, or designing responsive interfaces. Triggers on Tailwind, CSS, styling, responsive, animation, dark mode, or design keywords.
---

# Tailwind CSS v4 Modern Patterns

## Design Philosophy

**Avoid generic "AI slop" aesthetics.** Create distinctive, memorable interfaces.

### Key Principles

1. **Bold choices** over safe defaults
2. **Generous whitespace** for breathing room
3. **Cohesive color palettes** with sharp accents
4. **Subtle animations** that delight
5. **Mobile-first** responsive design

---

## Tailwind v4 Setup

### CSS-First Configuration

Tailwind v4 uses `@theme` directive in CSS instead of `tailwind.config.ts`:

```css
/* globals.css */
@import "tailwindcss";

@theme {
  /* Colors - using oklch for better color mixing */
  --color-background: oklch(1 0 0);
  --color-foreground: oklch(0.145 0.04 264);
  --color-primary: oklch(0.6 0.25 264);
  --color-primary-foreground: oklch(0.985 0.002 264);
  --color-secondary: oklch(0.97 0.01 264);
  --color-muted: oklch(0.97 0.01 264);
  --color-muted-foreground: oklch(0.556 0.02 264);
  --color-accent: oklch(0.97 0.01 264);
  --color-destructive: oklch(0.577 0.245 27);
  --color-border: oklch(0.922 0.01 264);
  --color-ring: oklch(0.6 0.25 264);

  /* Typography */
  --font-sans: "Inter", system-ui, sans-serif;
  --font-display: "Cal Sans", "Inter", sans-serif;
  --font-mono: "JetBrains Mono", monospace;

  /* Spacing & Sizing */
  --radius: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;

  /* Custom breakpoints */
  --breakpoint-3xl: 120rem;

  /* Easing functions */
  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
  --ease-snappy: cubic-bezier(0.2, 0, 0, 1);

  /* Custom animations */
  --animate-fade-in: fade-in 0.3s var(--ease-fluid);
  --animate-slide-up: slide-up 0.4s var(--ease-fluid);
  --animate-slide-down: slide-down 0.4s var(--ease-fluid);
  --animate-scale-in: scale-in 0.2s var(--ease-snappy);
}

/* Dark mode theme */
@theme dark {
  --color-background: oklch(0.145 0.04 264);
  --color-foreground: oklch(0.985 0.002 264);
  --color-primary: oklch(0.7 0.25 264);
  --color-primary-foreground: oklch(0.145 0.04 264);
  --color-secondary: oklch(0.269 0.03 264);
  --color-muted: oklch(0.269 0.03 264);
  --color-muted-foreground: oklch(0.708 0.02 264);
  --color-accent: oklch(0.269 0.03 264);
  --color-border: oklch(0.269 0.03 264);
}

/* Keyframes */
@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes slide-up {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes slide-down {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes scale-in {
  from { opacity: 0; transform: scale(0.95); }
  to { opacity: 1; transform: scale(1); }
}
```

### Using Theme Variables

```tsx
// Direct usage in classes
<div className="bg-background text-foreground" />
<button className="bg-primary text-primary-foreground" />
<p className="text-muted-foreground" />
```

---

## Typography

### Text Hierarchy

```tsx
// Hero title
className="text-4xl md:text-5xl lg:text-6xl font-bold tracking-tight"

// Section title
className="text-2xl md:text-3xl font-semibold tracking-tight"

// Card title
className="text-lg font-semibold"

// Body
className="text-base text-muted-foreground leading-relaxed"

// Small / Caption
className="text-sm text-muted-foreground"

// Display font
className="font-display text-4xl font-bold tracking-tight"

// Monospace
className="font-mono text-sm"
```

---

## Spacing & Layout

### Container

```tsx
<div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
  {/* Content */}
</div>
```

### Section Spacing

```tsx
// Page sections
<section className="py-16 md:py-24 lg:py-32">

// Cards
<div className="p-6 md:p-8">

// Compact elements
<div className="p-4">
```

### Grid Patterns

```tsx
// Auto-fit responsive grid
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">

// Sidebar layout
<div className="grid lg:grid-cols-[280px_1fr] gap-8">

// Dashboard grid
<div className="grid grid-cols-2 md:grid-cols-4 gap-4">

// CSS Grid with arbitrary values
<div className="grid grid-cols-[24rem_2.5rem_minmax(0,1fr)]">
```

---

## Component Patterns

### Cards

```tsx
// Basic card
<div className="rounded-xl border border-border bg-background p-6 shadow-sm">

// Elevated card
<div className="rounded-2xl bg-gradient-to-br from-secondary to-muted
                p-6 shadow-lg shadow-border/50 border border-border/50">

// Interactive card
<div className="rounded-xl border border-border bg-background p-6
                transition-all duration-200
                hover:shadow-lg hover:border-primary/20
                hover:-translate-y-0.5">

// Glass card
<div className="rounded-xl backdrop-blur-md bg-background/70
                border border-border/20 shadow-xl p-6">
```

### Buttons

```tsx
// Primary
<button className="inline-flex items-center justify-center
                   rounded-lg bg-primary px-4 py-2
                   text-sm font-medium text-primary-foreground
                   shadow-sm transition-colors
                   hover:bg-primary/90
                   focus-visible:outline-none focus-visible:ring-2
                   focus-visible:ring-ring focus-visible:ring-offset-2
                   disabled:pointer-events-none disabled:opacity-50">

// Secondary
<button className="inline-flex items-center justify-center
                   rounded-lg border border-border bg-background
                   px-4 py-2 text-sm font-medium
                   shadow-sm transition-colors
                   hover:bg-accent hover:text-foreground">

// Ghost
<button className="inline-flex items-center justify-center
                   rounded-lg px-4 py-2 text-sm font-medium
                   transition-colors
                   hover:bg-accent hover:text-foreground">

// Icon button
<button className="inline-flex size-10 items-center justify-center
                   rounded-lg transition-colors hover:bg-accent">
```

### Inputs

```tsx
// Text input
<input className="flex h-10 w-full rounded-lg border border-border
                  bg-background px-3 py-2 text-sm
                  ring-offset-background
                  placeholder:text-muted-foreground
                  focus-visible:outline-none focus-visible:ring-2
                  focus-visible:ring-ring focus-visible:ring-offset-2
                  disabled:cursor-not-allowed disabled:opacity-50" />

// With icon
<div className="relative">
  <SearchIcon className="absolute left-3 top-1/2 size-4
                         -translate-y-1/2 text-muted-foreground" />
  <input className="pl-10 ..." />
</div>
```

---

## Animations

### CSS Transitions

```tsx
// Hover lift
className="transition-all duration-200 hover:-translate-y-1 hover:shadow-lg"

// Scale on hover
className="transition-transform duration-200 hover:scale-105"

// Color transition
className="transition-colors duration-200 hover:bg-primary/90"

// Combined with custom easing
className="transition-all duration-300 ease-[--ease-fluid]
           hover:shadow-xl hover:-translate-y-0.5 hover:border-primary/20"
```

### Using Theme Animations

```tsx
// Fade in
<div className="animate-fade-in" />

// Slide up
<div className="animate-slide-up" />

// Scale in
<div className="animate-scale-in" />
```

### Staggered Animations

```tsx
<ul>
  {items.map((item, i) => (
    <li
      key={item.id}
      className="animate-slide-up opacity-0"
      style={{
        animationDelay: `${i * 100}ms`,
        animationFillMode: 'forwards'
      }}
    >
      {item.name}
    </li>
  ))}
</ul>
```

---

## Responsive Design

### Breakpoints

| Prefix | Min Width | Typical Use |
|--------|-----------|-------------|
| `sm:` | 640px | Large phones |
| `md:` | 768px | Tablets |
| `lg:` | 1024px | Laptops |
| `xl:` | 1280px | Desktops |
| `2xl:` | 1536px | Large screens |
| `3xl:` | 1920px | Custom (defined in @theme) |

### Mobile-First Pattern

```tsx
// Stack on mobile, row on desktop
<div className="flex flex-col md:flex-row gap-4">

// Full width mobile, constrained desktop
<div className="w-full max-w-md mx-auto lg:max-w-none lg:mx-0">

// Different padding per breakpoint
<div className="p-4 sm:p-6 lg:p-8">

// Hide/show elements
<div className="hidden md:block">Desktop only</div>
<div className="md:hidden">Mobile only</div>
```

---

## Dark Mode

### Setup

Dark mode in v4 uses `@theme dark` block (see setup above).

### Pattern

```tsx
// Colors automatically switch with dark class
<div className="bg-background text-foreground border-border">
  {/* Automatically adapts to dark mode */}
</div>

// Manual dark mode overrides when needed
<div className="bg-white dark:bg-slate-900
                text-slate-900 dark:text-slate-100">
```

### Toggle Component

```tsx
'use client'
import { useTheme } from 'next-themes'
import { Moon, Sun } from 'lucide-react'

export function ThemeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <button
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
      className="relative rounded-lg p-2 hover:bg-accent"
    >
      <Sun className="size-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute left-2 top-2 size-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
    </button>
  )
}
```

---

## Accessibility

```tsx
// Focus visible (keyboard only)
className="focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"

// Screen reader only
className="sr-only"

// Reduced motion
className="motion-reduce:transition-none motion-reduce:animate-none"

// Disabled state
className="disabled:pointer-events-none disabled:opacity-50"

// Focus within (for form groups)
className="focus-within:ring-2 focus-within:ring-ring"
```

---

## Arbitrary Values & CSS Variables

```tsx
// Arbitrary colors
<button className="bg-[#316ff6]">Sign in with Facebook</button>

// Arbitrary grid
<div className="grid grid-cols-[24rem_2.5rem_minmax(0,1fr)]">

// Using calc with CSS variables
<div className="max-h-[calc(100dvh-(--spacing(6)))]">

// Custom CSS variables in classes
<div className="[--gutter-width:1rem] lg:[--gutter-width:2rem]">
  <div className="px-[--gutter-width]">Content</div>
</div>
```

---

## Common Utilities

```tsx
// Truncate text
className="truncate"           // Single line with ellipsis
className="line-clamp-2"       // Multi-line with ellipsis

// Aspect ratios
className="aspect-video"       // 16:9
className="aspect-square"      // 1:1

// Size shorthand (width + height)
className="size-10"            // w-10 h-10

// Object fit
className="object-cover"       // Cover container
className="object-contain"     // Fit inside

// Scrolling
className="overflow-y-auto"    // Vertical scroll
className="overscroll-contain" // Prevent scroll chaining

// Selection
className="select-none"        // Prevent selection
className="cursor-pointer"     // Pointer cursor
```

---

## Migration from v3

| v3 Pattern | v4 Pattern |
|------------|------------|
| `tailwind.config.ts` colors | `@theme { --color-* }` |
| `h-10 w-10` | `size-10` |
| `ring-offset-2` | `ring-offset-2` (unchanged) |
| Custom plugins | CSS `@theme` or `@utility` |
