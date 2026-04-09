---
name: s-pacine-design
description: Use when building new pages or components — provides the color palette, typography, spacing, and component patterns for Pa' Cine
---

# Pa' Cine Design System

Follow these colors, typography, spacing, and patterns exactly to stay visually consistent with the existing app.

## Tech Stack

- Next.js 15 (App Router) + TypeScript
- Tailwind CSS 3 with utility-first approach
- shadcn/ui components (built on Radix UI)
- lucide-react icons
- Font: Inter (weights 400, 500, 600, 700, 800)

## Color Palette

### Primary -- Red (brand color)

| Token               | Hex       | Usage                                      |
|----------------------|-----------|---------------------------------------------|
| `red-500`            | `#ef4444` | Logo accent, active tab underline, spinners, primary buttons, hover text |
| `red-600`            | `#dc2626` | Pressed/darker primary states               |

### Secondary -- Amber (accents, ratings, selected states)

| Token               | Hex       | Usage                                    |
|----------------------|-----------|------------------------------------------|
| `amber-400` / `secondary-400` | `#fbbf24` | Selected states, accent highlights   |
| `amber-500`          | `#f59e0b` | Ratings, badges                         |
| `amber-600`          | `#d97706` | Format labels (uppercase text)          |

### Neutrals -- Zinc

| Token         | Hex       | Usage                                         |
|---------------|-----------|------------------------------------------------|
| `zinc-50`     | `#fafafa` | Hover backgrounds (light)                      |
| `zinc-100`    | `#f4f4f5` | Page background (light mode)                   |
| `zinc-200`    | `#e4e4e7` | Borders, image placeholders, slider tracks     |
| `zinc-300`    | `#d4d4d8` | Inactive tab text, spinner border, borders     |
| `zinc-400`    | `#a1a1aa` | Footer text, muted icons, placeholder text     |
| `zinc-500`    | `#71717a` | Secondary text, summary counts                 |
| `zinc-700`    | `#3f3f46` | Section headings, dark borders                 |
| `zinc-800`    | `#27272a` | Menu hover backgrounds (dark)                  |
| `zinc-900`    | `#18181b` | Header/footer bg, body text, card text         |

### Semantic

| Token    | Hex       | Usage          |
|----------|-----------|----------------|
| `success`| `#10b981` | Success states  |
| `warning`| `#f97316` | Warnings        |
| `info`   | `#0ea5e9` | Info badges     |

### Colombian Flag (Footer heart)

| Color   | Hex       |
|---------|-----------|
| Yellow  | `#FCD116` |
| Blue    | `#003893` |
| Red     | `#CE1126` |

## Typography

- **Font family**: `'Inter', sans-serif` -- applied globally on `<body>`
- All headings (h1-h6) are `font-bold` by default via base layer
- **Brand title**: `text-xl font-black tracking-tight` (PA'CINE in header)
- **Tab labels**: `text-2xl font-black` with `border-b-[3px]`
- **Section headings**: `text-lg font-semibold`
- **Body text**: default size, `text-zinc-900`
- **Secondary text**: `text-sm text-zinc-500`
- **Card titles**: `font-semibold text-xs sm:text-base lg:text-lg`
- **Small labels**: `text-[10px] tracking-wider uppercase`

## Spacing & Layout

- **Page max-width**: `max-w-7xl mx-auto`
- **Page padding**: `px-4` (mobile), `md:px-4` (desktop)
- **Section gaps**: `mb-8` between sections, `mb-3` or `mb-4` for subsections
- **Card grid gaps**: `gap-3 sm:gap-4 lg:gap-6`
- **Consistent padding in cards**: `px-2 py-1 sm:px-4 sm:py-2`

## Component Patterns

### Cards

- Base class: `.card-base` -- `bg-white dark:bg-zinc-900 rounded-lg border border-zinc-200 dark:border-zinc-800`
- Hover: `hover:shadow-md transition-shadow`
- Poster aspect ratio: `aspect-[2/3]`
- Image hover: `hover:scale-105 transition-transform duration-300`

### Buttons

- shadcn button variants via `class-variance-authority`
- Primary: red-500 bg, white text
- Sizes: `h-8` (sm), `h-9` (default), `h-10` (lg)
- Always `rounded-md`

### Grids

- Movie grid: `grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6`
- Gap: `gap-3 sm:gap-4 lg:gap-6`

### Tabs

- Active: `text-zinc-900 border-red-500 border-b-[3px]`
- Inactive: `text-zinc-300 border-transparent hover:text-zinc-500`
- Font: `text-2xl font-black`

### Loading Spinner

```html
<div class="h-10 w-10 animate-spin rounded-full border-4 border-zinc-300 border-t-red-500" />
```

## Dark Mode

- Strategy: class-based (`dark:` prefix)
- Dark backgrounds: `dark:bg-zinc-900`
- Dark text: `dark:text-zinc-100`
- Dark borders: `dark:border-zinc-800`
- Brand colors (red, amber) stay the same in both modes

## General Rules

- All user-facing text in Spanish
- Locale: `es-CO` (Colombian Spanish)
- Mobile-first responsive design
- Transitions: `transition-colors`, `transition-shadow`, `transition-transform`
- Text overflow: `truncate` (single line) or `line-clamp-2` (multi line)
- Icons: lucide-react (`Menu`, `Film`, `Building2`, `MapPin`, `ChevronRight`, etc.)

## Files to Reference

- `Components.md` -- Reusable component snippets (Header, Footer, MovieCard, etc.)
- `Example.md` -- Simplified home page showing all patterns in one file
