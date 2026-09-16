# Design Guidelines

Complete design token system and visual guidelines for AI-generated prototypes.

## Design Philosophy

- **Clarity over cleverness** — Users should understand the interface instantly
- **Consistency breeds trust** — Repeat patterns, spacing, and colors throughout
- **Breathing room** — Whitespace is a design element, not wasted space
- **Progressive disclosure** — Show essential info first, reveal details on demand
- **Feedback is mandatory** — Every interaction gets a visible response

## Color System

### Primary Palette
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--color-primary` | `#4F46E5` | `#818CF8` | Primary buttons, active states, links |
| `--color-primary-hover` | `#4338CA` | `#6366F1` | Hover state for primary |
| `--color-primary-light` | `#EEF2FF` | `#312E81` | Primary backgrounds, badges |
| `--color-secondary` | `#0EA5E9` | `#38BDF8` | Secondary actions, accents |
| `--color-secondary-hover` | `#0284C7` | `#7DD3FC` | Hover state for secondary |

### Neutral Palette
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--color-bg` | `#FFFFFF` | `#0F172A` | Page background |
| `--color-surface` | `#F8FAFC` | `#1E293B` | Cards, panels, elevated surfaces |
| `--color-surface-hover` | `#F1F5F9` | `#334155` | Hover state for surfaces |
| `--color-text` | `#1E293B` | `#F1F5F9` | Primary text |
| `--color-text-muted` | `#64748B` | `#94A3B8` | Secondary text, labels |
| `--color-text-disabled` | `#94A3B8` | `#475569` | Disabled text |
| `--color-border` | `#E2E8F0` | `#334155` | Borders, dividers |

### Semantic Colors
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--color-success` | `#10B981` | `#34D399` | Success messages, positive indicators |
| `--color-success-bg` | `#D1FAE5` | `#064E3B` | Success background |
| `--color-warning` | `#F59E0B` | `#FBBF24` | Warning messages, caution |
| `--color-warning-bg` | `#FEF3C7` | `#78350F` | Warning background |
| `--color-error` | `#EF4444` | `#F87171` | Error messages, destructive actions |
| `--color-error-bg` | `#FEE2E2` | `#7F1D1D` | Error background |
| `--color-info` | `#3B82F6` | `#60A5FA` | Info messages, neutral notifications |
| `--color-info-bg` | `#DBEAFE` | `#1E3A8A` | Info background |

## Typography

### Font Stack
```css
--font-sans: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
--font-mono: 'JetBrains Mono', 'Fira Code', 'Courier New', monospace;
```

### Type Scale
| Token | Size | Line Height | Weight | Usage |
|-------|------|-------------|--------|-------|
| `--text-xs` | 12px | 16px | 400 | Labels, captions, metadata |
| `--text-sm` | 14px | 20px | 400 | Secondary text, table cells |
| `--text-base` | 16px | 24px | 400 | Body text, default |
| `--text-lg` | 18px | 28px | 400 | Emphasized body text |
| `--text-xl` | 20px | 28px | 600 | Card titles, section headers |
| `--text-2xl` | 24px | 32px | 600 | Page section titles |
| `--text-3xl` | 30px | 36px | 700 | Page titles |
| `--text-4xl` | 36px | 40px | 700 | Hero headlines |
| `--text-5xl` | 48px | 56px | 800 | Large hero headlines |

## Spacing System

Based on a 4px grid:
```
0, 4px, 8px, 12px, 16px, 20px, 24px, 32px, 40px, 48px, 64px, 80px, 96px, 128px
```

| Token | Value | Usage |
|-------|-------|-------|
| `--space-xs` | 4px | Tight spacing between related elements |
| `--space-sm` | 8px | Icon-text gaps, small padding |
| `--space-md` | 16px | Default element spacing, card padding |
| `--space-lg` | 24px | Section internal spacing |
| `--space-xl` | 32px | Between sections |
| `--space-2xl` | 48px | Major section breaks |
| `--space-3xl` | 64px | Page-level vertical rhythm |

## Border Radius
| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | 4px | Small elements, badges |
| `--radius-md` | 8px | Buttons, inputs, cards |
| `--radius-lg` | 12px | Large cards, modals |
| `--radius-xl` | 16px | Feature sections |
| `--radius-full` | 9999px | Pills, avatars, circular buttons |

## Shadows
| Token | Value | Usage |
|-------|-------|-------|
| `--shadow-xs` | `0 1px 2px rgba(0,0,0,0.05)` | Subtle elevation (inputs) |
| `--shadow-sm` | `0 1px 3px rgba(0,0,0,0.1), 0 1px 2px rgba(0,0,0,0.06)` | Cards, dropdowns |
| `--shadow-md` | `0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -1px rgba(0,0,0,0.06)` | Hovered cards, popovers |
| `--shadow-lg` | `0 10px 15px -3px rgba(0,0,0,0.1), 0 4px 6px -2px rgba(0,0,0,0.05)` | Modals, floating elements |
| `--shadow-xl` | `0 20px 25px -5px rgba(0,0,0,0.1), 0 10px 10px -5px rgba(0,0,0,0.04)` | Large modals |

## Transitions
| Token | Duration | Easing | Usage |
|-------|----------|--------|-------|
| `--transition-fast` | 150ms | `ease` | Hover states, small toggles |
| `--transition-base` | 200ms | `ease` | Default transitions |
| `--transition-slow` | 300ms | `ease` | Modals, panels, large elements |

## Layout Grid

### Desktop (1024px+)
- Max content width: 1200px (centered)
- Sidebar: 240px fixed
- Content area: flex-1
- Card grid: 3-4 columns (auto-fill, minmax 280px)

### Tablet (768px - 1023px)
- Max content width: 100%
- Sidebar: collapsible (64px collapsed, 240px expanded)
- Card grid: 2 columns

### Mobile (< 768px)
- Full width with 16px padding
- Sidebar: hidden, drawer overlay
- Card grid: 1 column
- Bottom navigation bar (max 5 items)

## Responsive Breakpoints
```css
/* Mobile first */
@media (min-width: 640px) { /* Small tablets */ }
@media (min-width: 768px) { /* Tablets */ }
@media (min-width: 1024px) { /* Desktop */ }
@media (min-width: 1280px) { /* Large desktop */ }
```

## Dark Mode Implementation

### Using CSS Media Query (Auto)
```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0F172A;
    --color-surface: #1E293B;
    --color-text: #F1F5F9;
    /* ... override all tokens */
  }
}
```

### Using Manual Toggle (Recommended)
```css
[data-theme="dark"] {
  --color-bg: #0F172A;
  --color-surface: #1E293B;
  --color-text: #F1F5F9;
  /* ... override all tokens */
}
```

JavaScript toggle:
```javascript
function toggleTheme() {
  const html = document.documentElement;
  const current = html.getAttribute('data-theme');
  html.setAttribute('data-theme', current === 'dark' ? 'light' : 'dark');
}
```

## Accessibility Checklist

- Color contrast ratio: at least 4.5:1 for normal text, 3:1 for large text
- Focus visible: all interactive elements have visible focus rings
- Keyboard navigation: Tab order is logical, no keyboard traps
- Alt text: all images have descriptive alt attributes
- ARIA labels: icon-only buttons have `aria-label`
- Semantic HTML: use proper landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`)
- Form labels: all inputs have associated `<label>` elements
