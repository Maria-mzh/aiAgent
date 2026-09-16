---
name: ai-prototype-design
description: "AI-driven interactive prototype design skill that generates clickable, responsive HTML/CSS/JS prototypes from natural language descriptions. This skill should be used when users want to create UI mockups, wireframes, interactive prototypes, product prototypes, app screens, dashboard layouts, landing pages, admin panels, or any visual interface design that can be validated through interaction. Trigger phrases include: generate a prototype, create a mockup, design a UI, make a wireframe, build a clickable prototype, create a dashboard, design a landing page, make an app screen, generate a wireframe, 生成原型, 制作原型, 设计原型, 生成界面, 创建交互原型, 做一个页面设计, 生成线框图, 设计仪表盘, 制作落地页, 设计后台管理界面, 生成App界面, 做一个原型图, 设计网页原型, 交互原型设计, 快速原型, 产品原型, UI原型, UX原型, 高保真原型, 低保真原型, clickable prototype, interactive mockup, product prototype, UI mockup, wireframe, hi-fi prototype, lo-fi prototype, prototype from description, text to prototype, design from requirements, PRD to prototype, 需求转原型, 描述生成页面. Also triggers on requests to iterate or modify existing prototypes: modify the prototype, update the layout, change the design, adjust colors, add a component, 修改原型, 调整布局, 更新设计, 添加组件. Covers both low-fidelity wireframes and high-fidelity interactive prototypes with realistic data, animations, and responsive behavior."
agent_created: true
---

# AI Prototype Design

Generate interactive, clickable HTML/CSS/JS prototypes from natural language descriptions — turning ideas into validated designs in minutes, not days.

## Overview

This skill transforms text descriptions into fully interactive HTML prototypes. Unlike static mockups, the output is real, runnable code that stakeholders can click through, test flows, and provide feedback on. The skill supports the full spectrum from low-fidelity wireframes to high-fidelity prototypes with realistic data, animations, and responsive layouts.

## When to Use

- **New product concepts** — "Help me design a task management app"
- **Feature validation** — "Create a prototype for a subscription checkout flow"
- **Landing pages** — "Design a SaaS landing page with hero, features, and pricing"
- **Dashboard/Admin** — "Build an analytics dashboard prototype"
- **Mobile screens** — "Design a mobile login and onboarding flow"
- **Wireframes** — "Create a low-fidelity wireframe for a blog platform"
- **Iteration** — "Modify the prototype: add a dark mode toggle"

## Workflow

### Step 1: Understand the Request

Parse the user's description to identify:

1. **Deliverable type** — landing page, dashboard, mobile app, admin panel, wireframe, or custom
2. **Fidelity level** — low-fidelity (wireframe, grayscale, placeholder content) vs. high-fidelity (realistic colors, data, interactions)
3. **Platform** — desktop, tablet, mobile, or responsive
4. **Key sections/screens** — what pages or views are needed
5. **Interaction requirements** — clickable navigation, form validation, modal dialogs, animations
6. **Style preferences** — minimalist, material, flat, glassmorphism, dark mode, brand colors

If the description is vague, ask 1-2 clarifying questions (max) before proceeding. Prefer making reasonable assumptions and noting them.

### Step 2: Select a Template (Optional Acceleration)

If the request matches a common pattern, start from a bundled template and customize:

| Request Pattern | Template File | When to Use |
|----------------|---------------|-------------|
| Dashboard / Analytics | `{SKILL_ROOT}/assets/templates/dashboard.html` | Data visualization, admin panels, analytics tools |
| Landing Page / Marketing | `{SKILL_ROOT}/assets/templates/landing-page.html` | SaaS, product launches, marketing sites |
| Mobile App | `{SKILL_ROOT}/assets/templates/mobile-app.html` | iOS/Android app screens, mobile-first design |
| Admin Panel / CRUD | `{SKILL_ROOT}/assets/templates/admin-panel.html` | Back-office, data management, settings pages |

**To use a template:** Read the template file, understand its structure, then modify it to match the user's requirements. Templates use CSS custom properties (variables) for easy theming.

If no template fits, build from scratch following the design guidelines in `{SKILL_ROOT}/references/design-guidelines.md`.

### Step 3: Design the Prototype

Follow these principles when generating the HTML/CSS/JS:

#### Structure
- Single self-contained HTML file (inline CSS and JS) for easy sharing
- Use semantic HTML5 elements (`<header>`, `<nav>`, `<main>`, `<section>`, `<footer>`)
- CSS custom properties for theming (colors, spacing, typography, shadows)
- Responsive breakpoints: mobile-first, then tablet (768px), then desktop (1024px)

#### Visual Design
- Load `{SKILL_ROOT}/references/design-guidelines.md` for detailed design tokens and patterns
- Default to a clean, modern aesthetic unless specified otherwise
- Use a consistent spacing scale (4px base: 4, 8, 12, 16, 24, 32, 48, 64)
- Typography hierarchy: at least 3 levels (heading, subheading, body)
- Color palette: primary, secondary, neutral grays, semantic colors (success, warning, error, info)
- Use subtle shadows and rounded corners for depth (not flat, not skeuomorphic)

#### Interactivity
- Navigation between screens/views (show/hide sections or multi-page)
- Hover states on all interactive elements
- Form inputs with focus states
- At least one modal/dialog or dropdown interaction
- Smooth transitions (200-300ms ease) on state changes
- Tab switching, accordion expansion, or similar micro-interactions

#### Content
- Use realistic placeholder data (not "Lorem Ipsum" — use meaningful sample content)
- Include realistic user names, product names, dates, numbers
- For dashboards: include charts using inline SVG or simple CSS bar charts
- For lists/tables: include 5-10 rows of sample data

### Step 4: Validate and Refine

Before presenting the prototype, verify:

1. **Self-contained** — No external CDN dependencies that could break offline (except Google Fonts)
2. **Responsive** — Layout works on mobile (375px), tablet (768px), and desktop (1024px+)
3. **Clickable** — All navigation links work, buttons have click handlers
4. **Accessible** — Sufficient color contrast, alt text on images, keyboard-navigable
5. **Performance** — No heavy frameworks, vanilla JS only, minimal DOM manipulation

### Step 5: Present and Iterate

1. Save the HTML file to the output directory
2. Open it in the browser preview for the user
3. Ask for feedback and offer to iterate on specific sections
4. Common iterations: adjust colors, add screens, modify layout, add dark mode, change content

## Design Token Reference

Quick reference for default design tokens. Full details in `{SKILL_ROOT}/references/design-guidelines.md`.

### Colors (Light Theme)
```
--color-primary: #4F46E5 (Indigo)
--color-primary-hover: #4338CA
--color-secondary: #0EA5E9 (Sky Blue)
--color-bg: #FFFFFF
--color-surface: #F8FAFC
--color-text: #1E293B
--color-text-muted: #64748B
--color-border: #E2E8F0
--color-success: #10B981
--color-warning: #F59E0B
--color-error: #EF4444
```

### Spacing
```
--space-xs: 4px
--space-sm: 8px
--space-md: 16px
--space-lg: 24px
--space-xl: 32px
--space-2xl: 48px
--space-3xl: 64px
```

### Typography
```
--font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif
--text-xs: 12px
--text-sm: 14px
--text-base: 16px
--text-lg: 18px
--text-xl: 20px
--text-2xl: 24px
--text-3xl: 30px
--text-4xl: 36px
```

### Shadows
```
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05)
--shadow-md: 0 4px 6px -1px rgba(0,0,0,0.1)
--shadow-lg: 0 10px 15px -3px rgba(0,0,0,0.1)
```

## Component Patterns

The skill uses a consistent set of UI component patterns. Reference `{SKILL_ROOT}/references/component-patterns.md` for implementation details of:

- **Navigation bars** — top nav, sidebar, bottom tab bar (mobile)
- **Cards** — content cards, stat cards, profile cards
- **Forms** — input fields, selects, checkboxes, radio buttons, toggles
- **Data display** — tables, lists, badges, avatars, progress bars
- **Feedback** — toasts, modals, alerts, loading spinners
- **Charts** — bar charts, line charts, donut charts (pure CSS/SVG)

## Fidelity Levels

### Low-Fidelity Wireframe
- Grayscale color palette (blacks, grays, whites)
- Placeholder boxes for images (with X marks or labels)
- Minimal styling, focus on layout and structure
- Lorem-style placeholder text is acceptable here
- No animations or transitions

### Medium-Fidelity
- Limited color palette (1-2 accent colors + neutrals)
- Basic typography hierarchy
- Simple icons (use Unicode symbols or inline SVG)
- Some interactive elements (navigation, tabs)

### High-Fidelity
- Full color palette with semantic colors
- Realistic content and data
- Detailed icons and illustrations (inline SVG)
- Full interactivity: hover states, transitions, modals
- Micro-animations and polish
- Responsive behavior

## Output Format

All prototypes are delivered as a single HTML file:

```
prototype-name.html
├── <head>
│   ├── Meta tags (viewport, charset)
│   ├── Google Fonts link
│   └── <style> (all CSS inline)
├── <body>
│   ├── HTML structure
│   └── <script> (all JS inline at end of body)
```

File naming: `prototype-{type}-{date}.html` (e.g., `prototype-dashboard-20260720.html`)

## Common Patterns by Request Type

### Landing Page
1. Hero section (headline, subheadline, CTA button, hero image/illustration)
2. Social proof (logos, testimonials)
3. Features section (3-4 feature cards with icons)
4. How it works (3-step process)
5. Pricing (2-3 tier cards)
6. FAQ (accordion)
7. CTA section
8. Footer

### Dashboard
1. Sidebar navigation (collapsible on mobile)
2. Top bar (search, notifications, user menu)
3. KPI cards row (4 metric cards with trends)
4. Main chart area (bar/line chart)
5. Secondary widgets (recent activity, task list, quick stats)
6. Data table with pagination

### Mobile App
1. Status bar simulation
2. App header (title, back button, action button)
3. Content area (scrollable)
4. Bottom tab bar (3-5 tabs)
5. Floating action button (if applicable)
6. Modal sheets for actions

### Admin Panel
1. Sidebar with nested menu
2. Breadcrumb navigation
3. Filter/search bar
4. Data table with sortable columns
5. Bulk actions toolbar
6. Pagination and page size selector
7. Detail drawer/modal on row click

## Iteration Guidelines

When the user requests changes to an existing prototype:

1. **Read the existing file** first to understand current structure
2. **Make targeted edits** — don't regenerate the entire file for small changes
3. **Preserve existing styling** unless the change is explicitly about restyling
4. **Test after changes** — ensure navigation and interactions still work
5. **Common iterations:**
   - Color/theme changes → update CSS custom properties
   - Add a new section → insert HTML + corresponding CSS
   - Add a new page/screen → add section + navigation link
   - Change layout → modify CSS grid/flexbox properties
   - Add dark mode → add `prefers-color-scheme` media query or toggle

## Best Practices

- **Single file output** — Everything inline for portability and easy sharing
- **No build tools** — Vanilla HTML/CSS/JS, no npm, no bundlers, no frameworks
- **Google Fonts only** — Use Inter, Roboto, or system fonts; avoid obscure fonts
- **Inline SVG for icons** — No icon font libraries; use simple SVG paths
- **CSS Grid + Flexbox** — Modern layout methods, no float-based layouts
- **Semantic HTML** — Proper tags for accessibility and SEO
- **Print-friendly** — Include basic print styles for dashboards and reports
