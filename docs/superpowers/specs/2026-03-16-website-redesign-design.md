# Website Redesign — Design Spec
**Date:** 2026-03-16
**Status:** Approved

---

## Overview

Rebuild Ruotong Wang's personal academic website using the "Academic Personal Website" React template as the base. The template source is located at `Academic Personal Website/` inside the same repository root. Port all existing content, fix three limitations in the template, and add a Notion-friendly blog workflow. The result is a single-page academic portfolio with an attached writing/blog section.

---

## Stack

| Layer | Choice |
|---|---|
| Framework | React + Vite + TypeScript |
| Styling | Tailwind CSS |
| Routing | React Router |
| Blog content | MDX via `@mdx-js/rollup` |
| Post discovery | `import.meta.glob` (no manual registry) |
| Deployment | Vercel (auto-deploy on push to `main`) |

---

## Pages

Three routes; no About page.

| Route | Page | Nav |
|---|---|---|
| `/` | Home | None (no top nav bar) |
| `/writing` | Writing index | `← back` breadcrumb to `/` |
| `/writing/:slug` | Blog post | `← back` breadcrumb to `/writing` |

---

## File Structure

```
personal-website/
  src/
    app/
      components/
        Layout.tsx              — removed; no shared layout (each page is self-contained)
        figma/ImageWithFallback.tsx
        ui/                     — shadcn components (unchanged)
      pages/
        Home.tsx                — all content, no top nav
        Writing.tsx             — EB Garamond/DM Sans, ← back breadcrumb
        Post.tsx                — EB Garamond/DM Sans, MDX renderer, ← back breadcrumb
      routes.tsx                — remove /about route
      theme-context.ts
    content/
      posts/                    — MDX blog posts
        placeholder.mdx         — formatted example post
    styles/
      fonts.css                 — add EB Garamond + DM Sans; keep Manrope
      index.css
      tailwind.css
      theme.css                 — keep template color palette unchanged
  public/
    assets/
      images/
        profile-pic.jpeg        — copied from current site
        papers/                 — paper thumbnail images (optional, falls back to gradient)
      papers/                   — paper PDFs (copied from current site)
      cv/
        CV-2025-03.pdf          — copied from current site
  docs/
    superpowers/specs/          — this file
```

---

## Color Palette

Keep the template's palette unchanged throughout all pages (home + blog).

| Token | Value | Usage |
|---|---|---|
| `--background` | `#fdfbf7` | Page background |
| `--foreground` | `#433d3b` | Body text |
| `--primary` | `#c27a6e` | Links, accents, award badges |
| `--muted-foreground` | `#8a7d76` | Secondary text, dates, labels |
| `--border` | `#e5decb` | Dividers, thumbnail borders |
| `--muted` | `#eae4d9` | Placeholder backgrounds |

Dark mode toggle is preserved from the template.

---

## Typography

### Home page
- **Body / UI:** Manrope (sans-serif) — same as template default

### Writing index + Blog post pages
- **Body text:** EB Garamond (serif) — warm, scholarly, readable at size 18px
- **Headings, dates, labels, UI elements:** DM Sans (sans-serif) — clean, neutral

Both fonts loaded from Google Fonts. Applied via a `.blog-page` CSS class scoped to Writing and Post pages so home page typography is unaffected.

---

## Home Page Layout

### Structure
- No top navigation bar
- Centered container: `max-width: 860px`, horizontally centered, `padding: 48px 32px 64px`
- Two-column CSS grid: `200px sidebar | 1fr main`
- Sidebar is `position: sticky; top: 2rem` on desktop

### Left Sidebar (200px)
1. **Profile photo** — square, `border-radius: 4px`, loads from `/assets/images/profile-pic.jpeg`
2. **Name** — "Ruotong Wang", 14px bold
3. **Email** — ruotongw@cs.washington.edu, 10px muted
4. **Links** (with `→` arrow in primary color):
   - C.V. → `/assets/cv/CV-2025-03.pdf`
   - Google Scholar → external
   - Twitter → external
   - GitHub → external
5. **News section** (label + items, compact)
   - Section label: 9px uppercase, border-bottom
   - Items: `<strong>Month Year</strong> text`, 10px, 11px line-height
   - Show all news items from old site
6. **Writing section** (label + recent posts + "All writing →")
   - Show up to 2 most recent post titles with date; if fewer than 2 posts exist, show only what is available; if no posts exist, show only the "All writing →" link (no empty list)
   - "All writing →" link to `/writing`

### Right Main Column
1. **Bio** — two paragraphs ported from current site, 13px, line-height 1.9, `margin-bottom: 52px`
2. **Publications heading** — 9px uppercase label, `margin-bottom: 32px`
3. **Publication list** — all 12 papers, `gap: 44px` between entries, no divider lines

### Publication Entry
Each entry: `display: flex; gap: 20px`
- **Thumbnail** (80×60px, `border-radius: 3px`, `border: 1px solid #e5decb`):
  - Loads from `/assets/images/papers/<slug>.png` if available
  - Falls back to a soft gradient placeholder (unique color per paper)
- **Body:**
  - Title: 13px bold
  - Authors: 11px muted (first author name in foreground weight)
  - Venue: 11px muted
  - Award badge (if applicable): `🏆 Best Paper Honorable Mention`, terracotta pill
  - Links: `[arXiv]`, `[PDF]`, `[ACM DL]`, `[Video]` etc. in 10px monospace primary color

### All 12 Publications (in order, newest first)

Full titles, authors, venue, and links are sourced from `index.html` in the current site root. Implementer should copy them verbatim.
1. Social-RAG — CHI 2025
2. Meeting Bridges — CSCW 2024
3. Investigating and designing for trust in AI-powered code generation tools — FAccT 2024
4. "Is Reporting Worth the Sacrifice..." — SOUPS 2023
5. Personalizing content moderation on social media — CSCW 2023
6. "It would work for me too" — TiiS 2023
7. How Do Data Science Workers Communicate Intermediate Results? — VDS 2022
8. Tabletop Games in the Age of Remote Collaboration — CHI 2021 *(Best Paper HM)*
9. Decolonial Pathways — alt.chi 2021
10. Factors Influencing Perceived Fairness... — CHI 2020
11. Explaining Decision-Making Algorithms Through UI — CHI 2019
12. Exploring Interactions with Voice-Controlled TV — arXiv 2019

---

## Writing Index Page (`/writing`)

- `← back` breadcrumb at top (DM Sans, muted, links to `/`)
- Page title: "Writing" in EB Garamond 32px
- Subtitle: italic EB Garamond, muted
- Posts grouped by year (year label on left, posts on right — same grid as template)
- Post titles in EB Garamond with underline hover; dates in DM Sans

Post list is dynamically built from `import.meta.glob('../../content/posts/*.mdx')` sorted by frontmatter `date` descending.

---

## Blog Post Page (`/writing/:slug`)

- `← back` breadcrumb links to `/writing`
- Title: EB Garamond 36px bold
- Date: DM Sans 11px uppercase muted
- Body: EB Garamond 18px, line-height 1.8
- Headings (h2, h3): DM Sans bold
- Supports MDX features: bold, italic, inline code, code blocks, blockquotes, links

### Post Page Layout

- Outer container: `max-width: 960px`, horizontally centered, same padding as home page
- On **≥ 1024px**: two-column flex row — main content (`flex: 1`, max-width ~680px) + sticky TOC sidebar (`width: 200px`, `position: sticky; top: 2rem`)
- On **< 1024px**: single column; TOC sidebar is hidden entirely
- TOC sidebar: DM Sans 13px, lists all `h2` headings in the post, highlights the active heading via IntersectionObserver (same pattern as template's existing `Post.tsx`)

---

## MDX Post Format

Each post is an `.mdx` file in `src/content/posts/`:

```mdx
---
title: "Your Post Title"
date: "2026-03-15"
description: "One sentence summary."
---

Body content goes here. Export from Notion as Markdown, paste below the frontmatter.
```

**Notion → Blog workflow:**
1. Write in Notion
2. Export page as Markdown (`Export → Markdown & CSV`)
3. Create `src/content/posts/your-title.mdx`
4. Paste 4-line frontmatter block at top, paste Notion markdown below
5. `git add` → `git commit` → `git push`
6. Vercel deploys automatically (~30 seconds)

The placeholder post at `src/content/posts/placeholder.mdx` demonstrates all supported formatting: headings, paragraphs, bold/italic, blockquote, inline code, code block.

---

## Routing Changes from Template

| Template route | New site |
|---|---|
| `/` | Home (unchanged path, new content) |
| `/about` | **Removed** |
| `/writing` | Writing index (unchanged path) |
| `/writing/:slug` | Post (unchanged path, now MDX-driven) |

Shared `Layout` wrapper component is removed. Each page manages its own top-level structure. Dark mode context (`theme-context.ts`) is kept.

### Footer (Home page only)

A minimal footer sits at the bottom of the home page below the publications list:
- Left: `© 2026 Ruotong Wang`
- Right: dark/light mode toggle button (moon/sun icon, same as template)
- Separated from content by `border-top: 1px solid #e5decb`

The Writing index and Post pages have no footer. Navigation from those pages back to the home page is exclusively via the `← back` breadcrumb at the top.

---

## Responsive Behavior

Follows template's existing Tailwind breakpoints:
- **≥ 1024px (lg):** Two-column sidebar + main layout
- **< 1024px:** Single column; sidebar collapses above content, photo shrinks to 120px wide

---

## Assets to Copy from Current Site

| Source | Destination |
|---|---|
| `assets/images/profile-pic.jpeg` | `public/assets/images/profile-pic.jpeg` |
| `assets/papers/*.pdf` | `public/assets/papers/` |
| `assets/CV-2025-03.pdf` | `public/assets/cv/CV-2025-03.pdf` |

Paper thumbnail images are **not** required for launch — gradient placeholders display until images are added.

---

## Out of Scope

- About page (dropped)
- Separate projects page (dropped)
- Notion API integration (MDX workflow is sufficient)
- Comments or contact form
- Search
