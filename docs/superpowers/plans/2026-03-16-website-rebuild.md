# Personal Website Rebuild — Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild Ruotong Wang's personal academic website by modifying the React template in `Academic Personal Website/`, porting all content, adding MDX blog support, and deploying on Vercel.

**Architecture:** All work happens inside `Academic Personal Website/` (the existing React/Vite/Tailwind app). The shared `Layout` wrapper is removed; each page is self-contained. MDX powers blog posts via `@mdx-js/rollup` with frontmatter support. Publications are stored as a typed data array populated from Google Scholar. The old static site at the repo root is left untouched as a reference.

**Tech Stack:** React 18, Vite 6, TypeScript, Tailwind CSS v4, React Router 7, `@mdx-js/rollup`, `remark-frontmatter`, `remark-mdx-frontmatter`, Google Fonts (Manrope, EB Garamond, DM Sans), Vercel

**Spec:** `docs/superpowers/specs/2026-03-16-website-redesign-design.md`

---

## File Map

| Status | Path (relative to `Academic Personal Website/`) | Purpose |
|--------|--------------------------------------------------|---------|
| Modify | `vite.config.ts` | Add MDX plugin before React plugin |
| Modify | `src/styles/fonts.css` | Add EB Garamond + DM Sans |
| Modify | `src/styles/theme.css` | Add blog typography + prose CSS |
| Modify | `src/app/routes.tsx` | Remove About route, remove Layout wrapper |
| Modify | `src/app/App.tsx` | Restore localStorage theme on mount |
| Rewrite | `src/app/pages/Home.tsx` | Full home page — sidebar + bio + publications |
| Rewrite | `src/app/pages/Writing.tsx` | Blog index, MDX-powered, EB Garamond |
| Rewrite | `src/app/pages/Post.tsx` | Individual post, MDX renderer, h1/h2/h3 TOC |
| Create | `src/types/mdx.d.ts` | TypeScript declarations for `.mdx` modules |
| Create | `src/data/publications.ts` | All publications as typed array |
| Create | `src/app/hooks/usePosts.ts` | Shared post list utility |
| Create | `src/app/components/NoiseOverlay.tsx` | Shared noise texture overlay |
| Create | `src/content/posts/placeholder.mdx` | Example post showing full formatting |
| Create | `public/assets/images/profile-pic.jpeg` | Copied from old site |
| Create | `public/assets/cv/CV-2025-03.pdf` | Copied from old site |
| Create | `public/assets/papers/*.pdf` | Copied from old site |
| Create | `vercel.json` | SPA routing config for Vercel |

---

## Chunk 1: Foundation — MDX, fonts, routing, assets

### Task 1: Install MDX dependencies

**Files:**
- Modify: `Academic Personal Website/package.json` (via npm)

- [ ] **Step 1: Install packages**

```bash
cd "Academic Personal Website"
npm install @mdx-js/rollup remark-frontmatter remark-mdx-frontmatter
```

Expected: packages appear in `node_modules/`, `package.json` dependencies updated.

- [ ] **Step 2: Verify install**

```bash
npm list @mdx-js/rollup remark-frontmatter remark-mdx-frontmatter
```

Expected: all three listed with version numbers, no `UNMET DEPENDENCY` errors.

- [ ] **Step 3: Commit**

```bash
cd ..
git add "Academic Personal Website/package.json" "Academic Personal Website/package-lock.json"
git commit -m "feat: add MDX + remark frontmatter dependencies"
```

---

### Task 2: Configure Vite for MDX

**Files:**
- Modify: `Academic Personal Website/vite.config.ts`
- Create: `Academic Personal Website/src/types/mdx.d.ts`

- [ ] **Step 1: Update `vite.config.ts`**

Replace the entire file with:

```ts
import { defineConfig } from 'vite'
import path from 'path'
import tailwindcss from '@tailwindcss/vite'
import react from '@vitejs/plugin-react'
import mdx from '@mdx-js/rollup'
import remarkFrontmatter from 'remark-frontmatter'
import remarkMdxFrontmatter from 'remark-mdx-frontmatter'

export default defineConfig({
  plugins: [
    {
      enforce: 'pre',
      ...mdx({
        remarkPlugins: [remarkFrontmatter, remarkMdxFrontmatter],
      }),
    },
    react(),
    tailwindcss(),
  ],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  assetsInclude: ['**/*.svg', '**/*.csv'],
})
```

Note: MDX plugin must come **before** React with `enforce: 'pre'` to process `.mdx` files first.

- [ ] **Step 2: Create `src/types/mdx.d.ts`**

```ts
declare module '*.mdx' {
  import type { ComponentType } from 'react'

  export const frontmatter: {
    title: string
    date: string
    description: string
  }

  const Component: ComponentType
  export default Component
}
```

- [ ] **Step 3: Verify dev server starts without errors**

```bash
cd "Academic Personal Website"
npm run dev
```

Expected: server starts on `http://localhost:5173`, no errors in terminal.

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/vite.config.ts" "Academic Personal Website/src/types/"
git commit -m "feat: configure Vite MDX with frontmatter support"
```

---

### Task 3: Update fonts and add blog typography CSS

**Files:**
- Modify: `Academic Personal Website/src/styles/fonts.css`
- Modify: `Academic Personal Website/src/styles/theme.css`

- [ ] **Step 1: Update `src/styles/fonts.css`**

Replace the entire file with (single import line combining all fonts):

```css
@import url('https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400..700;1,400..700&family=Manrope:wght@200..800&family=EB+Garamond:ital,wght@0,400..700;1,400..700&family=DM+Sans:opsz,wght@9..40,300..700&display=swap');
```

- [ ] **Step 2: Add blog typography and prose CSS to `src/styles/theme.css`**

Append the following to the end of `src/styles/theme.css` (after the closing `}` of `@layer base`):

```css
/* ── Blog page typography ───────────────────────────────────────── */
/* Applied via .blog-page class on Writing and Post page roots */

.blog-page {
  font-family: 'EB Garamond', Georgia, serif;
}

.blog-page h1,
.blog-page h2,
.blog-page h3,
.blog-page h4,
.blog-ui {
  font-family: 'DM Sans', ui-sans-serif, system-ui, sans-serif;
}

/* ── MDX prose content ──────────────────────────────────────────── */
/* Applied via .prose-mdx class on the MDX content wrapper div */

.prose-mdx {
  font-family: 'EB Garamond', Georgia, serif;
  font-size: 18px;
  line-height: 1.8;
  color: var(--foreground);
}

.prose-mdx p {
  margin-bottom: 1.25em;
}

.prose-mdx h1,
.prose-mdx h2,
.prose-mdx h3 {
  font-family: 'DM Sans', ui-sans-serif, system-ui, sans-serif;
  font-weight: 700;
  color: var(--foreground);
  margin-top: 2em;
  margin-bottom: 0.75em;
  line-height: 1.3;
}

.prose-mdx h1 { font-size: 1.75em; }
.prose-mdx h2 { font-size: 1.375em; }
.prose-mdx h3 { font-size: 1.125em; }

.prose-mdx strong { font-weight: 700; }
.prose-mdx em { font-style: italic; }

.prose-mdx a {
  color: var(--primary);
  text-decoration: underline;
  text-underline-offset: 3px;
}

.prose-mdx blockquote {
  border-left: 3px solid var(--border);
  padding-left: 1.25em;
  color: var(--muted-foreground);
  font-style: italic;
  margin: 1.5em 0;
}

.prose-mdx code {
  font-family: ui-monospace, 'Cascadia Code', monospace;
  font-size: 0.85em;
  background: var(--muted);
  padding: 0.15em 0.35em;
  border-radius: 3px;
}

.prose-mdx pre {
  background: var(--muted);
  padding: 1.25em;
  border-radius: 6px;
  overflow-x: auto;
  margin: 1.5em 0;
  border: 1px solid var(--border);
}

.prose-mdx pre code {
  background: none;
  padding: 0;
  font-size: 0.875em;
}

.prose-mdx ul {
  list-style: disc;
  padding-left: 1.5em;
  margin-bottom: 1.25em;
}

.prose-mdx ol {
  list-style: decimal;
  padding-left: 1.5em;
  margin-bottom: 1.25em;
}

.prose-mdx li {
  margin-bottom: 0.4em;
}

.prose-mdx hr {
  border: none;
  border-top: 1px solid var(--border);
  margin: 2em 0;
}

.prose-mdx img {
  max-width: 100%;
  border-radius: 4px;
  margin: 1.5em 0;
}
```

- [ ] **Step 3: Verify fonts load**

Run dev server, open `http://localhost:5173`, open DevTools → Network → filter "Font". Confirm requests for `EB+Garamond` and `DM+Sans` appear.

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/styles/"
git commit -m "feat: add EB Garamond + DM Sans fonts, blog/prose CSS"
```

---

### Task 4: Simplify routing — remove About, remove Layout wrapper

**Files:**
- Modify: `Academic Personal Website/src/app/routes.tsx`

- [ ] **Step 1: Rewrite `src/app/routes.tsx`**

Replace the entire file with:

```ts
import { createBrowserRouter } from 'react-router'
import { Home } from './pages/Home'
import { Writing } from './pages/Writing'
import { Post } from './pages/Post'

export const router = createBrowserRouter([
  { path: '/', Component: Home },
  { path: '/writing', Component: Writing },
  { path: '/writing/:slug', Component: Post },
])
```

Note: `Layout` is no longer used as a wrapper. Each page manages its own full-page structure. `App.tsx` is unchanged — `ThemeProviderContext` already wraps `RouterProvider` there.

- [ ] **Step 2: Verify dev server still starts**

```bash
cd "Academic Personal Website"
npm run dev
```

Expected: server starts. Navigating to `http://localhost:5173` loads the (still-unchanged) Home component without errors. Console shows no missing component errors.

- [ ] **Step 3: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/routes.tsx"
git commit -m "feat: remove About route, remove shared Layout wrapper"
```

---

### Task 5: Copy static assets from old site

**Files:**
- Create: `Academic Personal Website/public/assets/images/profile-pic.jpeg`
- Create: `Academic Personal Website/public/assets/cv/CV-2025-03.pdf`
- Create: `Academic Personal Website/public/assets/papers/*.pdf`

All commands run from the repo root (`personal-website/`).

- [ ] **Step 1: Create asset directories**

```bash
mkdir -p "Academic Personal Website/public/assets/images/papers"
mkdir -p "Academic Personal Website/public/assets/papers"
mkdir -p "Academic Personal Website/public/assets/cv"
```

- [ ] **Step 2: Copy profile photo**

```bash
cp assets/images/profile-pic.jpeg "Academic Personal Website/public/assets/images/profile-pic.jpeg"
```

- [ ] **Step 3: Copy CV**

```bash
cp assets/CV-2025-03.pdf "Academic Personal Website/public/assets/cv/CV-2025-03.pdf"
```

- [ ] **Step 4: Copy paper PDFs**

```bash
cp assets/papers/*.pdf "Academic Personal Website/public/assets/papers/"
```

Expected: `ls "Academic Personal Website/public/assets/papers/"` lists the PDF files.

- [ ] **Step 5: Commit**

```bash
git add "Academic Personal Website/public/assets/"
git commit -m "feat: copy static assets (photo, CV, paper PDFs)"
```

---

## Chunk 2: Home Page

### Task 6: Fix App.tsx — restore theme from localStorage on mount

**Files:**
- Modify: `Academic Personal Website/src/app/App.tsx`

Currently `App.tsx` saves the theme to localStorage but initializes state as `"system"`, so the user's saved preference is lost on every page reload. Fix this.

- [ ] **Step 1: Read `src/app/App.tsx`**

- [ ] **Step 2: Replace the `useState` line**

Change:

```tsx
const [theme, setTheme] = useState<Theme>("system");
```

to:

```tsx
const [theme, setTheme] = useState<Theme>(() => {
  const stored = localStorage.getItem("vite-ui-theme") as Theme | null
  return stored ?? "system"
});
```

- [ ] **Step 3: Verify theme persists across reload**

Run dev server, toggle to dark mode, reload the page. Expected: dark mode is preserved after reload.

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/App.tsx"
git commit -m "fix: restore saved theme from localStorage on App mount"
```

---

### Task 7: Create usePosts shared hook

**Files:**
- Create: `Academic Personal Website/src/app/hooks/usePosts.ts`

This hook is used by both `Home.tsx` (Writing sidebar) and `Writing.tsx`, so it must exist before either page is implemented.

- [ ] **Step 1: Create `src/app/hooks/usePosts.ts`**

```ts
export interface PostMeta {
  slug: string
  title: string
  date: string
  description: string
}

/**
 * Returns all blog posts sorted by date descending (newest first).
 * Reads from src/content/posts/*.mdx via import.meta.glob.
 */
export function getPosts(): PostMeta[] {
  const modules = import.meta.glob('../../content/posts/*.mdx', {
    eager: true,
  }) as Record<
    string,
    { frontmatter: { title: string; date: string; description: string } }
  >

  return Object.entries(modules)
    .map(([filePath, mod]) => ({
      slug: filePath.split('/').pop()!.replace('.mdx', ''),
      title: mod.frontmatter.title,
      date: mod.frontmatter.date,
      description: mod.frontmatter.description,
    }))
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime())
}
```

Note: The glob path `../../content/posts/*.mdx` is relative to `src/app/hooks/`. It resolves to `src/content/posts/`. If `src/content/posts/` is empty, `getPosts()` returns `[]` — this is expected and handled gracefully by all consumers.

- [ ] **Step 2: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/hooks/usePosts.ts"
git commit -m "feat: add usePosts hook for MDX post discovery"
```

---

### Task 8: Build publications data file

**Files:**
- Create: `Academic Personal Website/src/data/publications.ts`

- [ ] **Step 1: Define the Publication interface**

Create `Academic Personal Website/src/data/publications.ts` with the interface only — the array is populated in Step 2 before any commit:

```ts
export interface PubLink {
  label: string  // e.g. 'arXiv', 'PDF', 'ACM DL', 'Video', 'USENIX', 'IEEE Xplore'
  url: string
}

export interface Publication {
  id: string           // kebab-case slug, used for thumbnail lookup
  title: string
  authors: string[]    // full author list; 'Ruotong Wang' will be bolded automatically
  venue: string        // e.g. 'CHI 2025', 'CSCW 2024'
  year: number
  award?: string       // e.g. 'Best Paper Honorable Mention'
  links: PubLink[]
}

export const publications: Publication[] = []
```

- [ ] **Step 2: Fetch Google Scholar and populate publications**

Open `https://scholar.google.com/citations?hl=en&user=CoG7C_YAAAAJ&view_op=list_works&sortby=pubdate` in a browser to get the full sorted publication list.

Cross-reference with `index.html` in the repo root for full author lists, venue details, and links (arXiv, PDF, ACM DL, Video, USENIX, etc.).

Replace the `publications` array with all papers, newest first. Example entry format:

```ts
{
  id: 'social-rag',
  title: 'Social-RAG: Retrieving from Group Interactions to Socially Ground AI Generation',
  authors: ['Ruotong Wang', 'Xinyi Zhou', 'Lin Qiu', 'Joseph Chee Chang', 'Jonathan Bragg', 'Amy Zhang'],
  venue: 'CHI 2025',
  year: 2025,
  links: [
    { label: 'arXiv', url: 'https://arxiv.org/abs/2411.02353' },
    { label: 'Video', url: 'https://www.loom.com/share/d4a7810a2a5142f7b46beaa7f382d0a9?sid=80f659dd-cfac-4964-bc44-c45bb4342862' },
  ],
},
```

Fill in all papers found on Google Scholar. For papers in `index.html` that have `href="#!"`, leave the URL as `'#'` as a placeholder.

- [ ] **Step 3: Verify TypeScript compiles**

```bash
cd "Academic Personal Website"
npm run build 2>&1 | head -30
```

Expected: build completes or fails only on unrelated issues (not type errors in `publications.ts`).

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/data/publications.ts"
git commit -m "feat: add publications data from Google Scholar"
```

---

### Task 9: Create NoiseOverlay shared component

**Files:**
- Create: `Academic Personal Website/src/app/components/NoiseOverlay.tsx`

- [ ] **Step 1: Create the component**

```tsx
export function NoiseOverlay() {
  return (
    <div
      className="pointer-events-none fixed inset-0 z-50 opacity-[0.03] dark:opacity-[0.04]"
      style={{
        backgroundImage: `url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)'/%3E%3C%2Fsvg%3E")`,
      }}
    />
  )
}
```

- [ ] **Step 2: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/components/NoiseOverlay.tsx"
git commit -m "feat: add NoiseOverlay shared component"
```

---

### Task 10: Implement Home.tsx

**Files:**
- Rewrite: `Academic Personal Website/src/app/pages/Home.tsx`

This is the largest component. Read the existing file first, then replace entirely.

- [ ] **Step 1: Read existing `Home.tsx`** to understand current imports.

- [ ] **Step 2: Write new `Home.tsx`**

Replace the entire file with:

```tsx
import { Link } from 'react-router'
import { Sun, Moon } from 'lucide-react'
import { useTheme } from '../theme-context'
import { publications } from '../../data/publications'
import { getPosts } from '../hooks/usePosts'
import { NoiseOverlay } from '../components/NoiseOverlay'
import { ImageWithFallback } from '../components/figma/ImageWithFallback'

const THUMB_GRADIENTS = [
  'linear-gradient(135deg, #e8ddd5, #d4c9be)',
  'linear-gradient(135deg, #d5dde8, #bec9d4)',
  'linear-gradient(135deg, #d5e8dd, #bec4be)',
  'linear-gradient(135deg, #e8e4d5, #d4cebe)',
  'linear-gradient(135deg, #ddd5e8, #c9bec9)',
  'linear-gradient(135deg, #e8d5d9, #d4bec2)',
]

const NEWS_ITEMS = [
  { date: 'November 2024', text: 'Attending CSCW in San José, Costa Rica.' },
  { date: 'October 2024', text: 'Serving as a Web Co-Chair for FAccT 2025.' },
  { date: 'June 2024', text: 'Attending CHIWORK in Newcastle upon Tyne, UK.' },
  { date: 'June 2024', text: 'Attending FAccT in Rio de Janeiro, Brazil.' },
  {
    date: 'June 2022',
    text: 'Starting summer internship at Microsoft Research, working with ',
    linkText: 'the SAINTES group',
    linkUrl: 'https://www.microsoft.com/en-us/research/group/saintes-group/',
    textAfter: '!',
  },
  { date: 'May 2022', text: 'Attending CHI in New Orleans.' },
  {
    date: 'Oct 2021',
    text: 'Attending CSCW virtually. Co-organizing a workshop on ',
    linkText: '"Subtle CSCW Traits"',
    linkUrl: 'https://subtlecscwtraits.wordpress.com/',
    textAfter: '.',
  },
  {
    date: 'June 2021',
    text: 'Starting summer internship at Microsoft Research, working with ',
    linkText: 'Dr. Mihaela Vorvoreanu',
    linkUrl: 'https://www.microsoft.com/en-us/research/people/mivorvor/',
    textAfter: '!',
  },
  { date: 'Oct 2020', text: 'Attending CSCW and UIST virtually and serving as a student volunteer for CSCW.' },
  { date: 'July 2020', text: 'Started (remotely) as a PhD student at UW.' },
  { date: 'Oct 2019', text: 'Moved to Pittsburgh and started working at HCII.' },
]

function formatShortDate(dateStr: string): string {
  const d = new Date(dateStr)
  return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
}

export function Home() {
  const { theme, setTheme } = useTheme()
  const recentPosts = getPosts().slice(0, 2)

  return (
    <div className="min-h-screen bg-background text-foreground transition-colors duration-500 selection:bg-primary/20 dark:selection:bg-primary/30">
      <NoiseOverlay />

      <div className="max-w-[860px] mx-auto px-8 py-12 pb-16 relative z-10">
        <div className="grid grid-cols-1 lg:grid-cols-[200px_1fr] gap-12 items-start">

          {/* ── Left Sidebar ── */}
          <aside className="lg:sticky lg:top-8">

            {/* Profile photo */}
            <div className="aspect-square w-full rounded overflow-hidden bg-muted border border-border mb-3">
              <ImageWithFallback
                src="/assets/images/profile-pic.jpeg"
                alt="Ruotong Wang"
                className="w-full h-full object-cover"
              />
            </div>

            {/* Name + email */}
            <p className="text-sm font-bold text-foreground mb-0.5">Ruotong Wang</p>
            <p className="text-xs text-muted-foreground mb-4">ruotongw@cs.washington.edu</p>

            {/* Links */}
            <div className="space-y-2 mb-0">
              {[
                { label: 'C.V.', href: '/assets/cv/CV-2025-03.pdf', external: true },
                { label: 'Google Scholar', href: 'https://scholar.google.com/citations?user=CoG7C_YAAAAJ&hl=en', external: true },
                { label: 'Twitter', href: 'https://twitter.com/RuotongWang1', external: true },
                { label: 'GitHub', href: 'https://github.com/Ruotongw', external: true },
              ].map((link) => (
                <a
                  key={link.label}
                  href={link.href}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="flex items-center gap-1.5 text-xs font-medium text-muted-foreground hover:text-primary transition-colors"
                >
                  {link.label}
                  <span className="text-primary text-[10px]">→</span>
                </a>
              ))}
            </div>

            {/* News section */}
            <div className="mt-7">
              <h3 className="text-[9px] font-bold uppercase tracking-widest text-muted-foreground border-b border-border pb-1.5 mb-3">
                News
              </h3>
              <ul className="space-y-2.5">
                {NEWS_ITEMS.map((item, i) => (
                  <li key={i} className="text-[10px] text-muted-foreground leading-relaxed">
                    <strong className="text-foreground">{item.date}</strong>{' '}
                    {item.linkText ? (
                      <>
                        {item.text}
                        <a
                          href={item.linkUrl}
                          target="_blank"
                          rel="noopener noreferrer"
                          className="text-primary hover:underline underline-offset-2"
                        >
                          {item.linkText}
                        </a>
                        {item.textAfter}
                      </>
                    ) : (
                      item.text
                    )}
                  </li>
                ))}
              </ul>
            </div>

            {/* Writing section */}
            <div className="mt-7">
              <h3 className="text-[9px] font-bold uppercase tracking-widest text-muted-foreground border-b border-border pb-1.5 mb-3">
                Writing
              </h3>
              {recentPosts.length > 0 && (
                <ul className="space-y-2.5 mb-3">
                  {recentPosts.map((post) => (
                    <li key={post.slug} className="flex items-baseline gap-1.5">
                      <span className="text-[9px] text-muted-foreground shrink-0">
                        {formatShortDate(post.date)}
                      </span>
                      <Link
                        to={`/writing/${post.slug}`}
                        className="text-[10px] text-foreground underline decoration-border underline-offset-2 hover:decoration-primary transition-colors leading-snug"
                      >
                        {post.title}
                      </Link>
                    </li>
                  ))}
                </ul>
              )}
              <Link
                to="/writing"
                className="text-[9px] font-bold uppercase tracking-wider text-primary hover:opacity-80 transition-opacity flex items-center gap-1"
              >
                All writing →
              </Link>
            </div>
          </aside>

          {/* ── Main Content ── */}
          <main className="space-y-0">

            {/* Bio */}
            <section className="mb-12 space-y-3">
              <p className="text-[13px] leading-[1.9] text-foreground">
                I'm a 5th year PhD student in Computer Science & Engineering at the University of
                Washington. I'm advised by{' '}
                <a
                  href="https://homes.cs.washington.edu/~axz/"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary hover:underline underline-offset-2"
                >
                  <strong>Amy Zhang</strong>
                </a>{' '}
                and part of the{' '}
                <a
                  href="http://social.cs.washington.edu/"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary hover:underline underline-offset-2"
                >
                  <strong>Social Futures Lab</strong>
                </a>
                . My research interests are human-computer interaction and social computing.
                Specifically, I am interested in designing interactive tools to support
                communication and collaboration in teams.
              </p>
              <p className="text-[13px] leading-[1.9] text-foreground">
                Before coming to UW, I spent a year in the Human-Computer Interaction Institute
                (HCII) at CMU, working with Prof.{' '}
                <a
                  href="http://haiyizhu.com/"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary hover:underline underline-offset-2"
                >
                  <strong>Haiyi Zhu</strong>
                </a>
                . Before that, I earned my BA at{' '}
                <a
                  href="http://macalester.edu/"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary hover:underline underline-offset-2"
                >
                  <strong>Macalester College</strong>
                </a>
                , where I studied Sociology, Mathematics. I also worked at the{' '}
                <a
                  href="https://grouplens.org/"
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary hover:underline underline-offset-2"
                >
                  <strong>GroupLens Lab</strong>
                </a>{' '}
                at the University of Minnesota during my undergrad.
              </p>
            </section>

            {/* Publications */}
            <section>
              <h2 className="text-[9px] font-bold uppercase tracking-widest text-muted-foreground mb-8">
                Publications
              </h2>
              <div className="space-y-11">
                {publications.map((pub, i) => (
                  <article key={pub.id} className="flex gap-5 items-start">

                    {/* Thumbnail */}
                    <div
                      className="w-20 h-[60px] shrink-0 rounded border border-border overflow-hidden relative"
                      style={{ background: THUMB_GRADIENTS[i % THUMB_GRADIENTS.length] }}
                    >
                      <img
                        src={`/assets/images/papers/${pub.id}.png`}
                        alt=""
                        className="absolute inset-0 w-full h-full object-cover"
                        onError={(e) => {
                          ;(e.target as HTMLImageElement).style.display = 'none'
                        }}
                      />
                    </div>

                    {/* Content */}
                    <div className="flex-1 space-y-1">
                      <p className="text-[13px] font-bold leading-snug text-foreground">
                        {pub.title}
                      </p>
                      <p className="text-[11px] text-muted-foreground leading-relaxed">
                        {pub.authors.map((author, ai) => (
                          <span key={ai}>
                            {ai > 0 && ', '}
                            {author === 'Ruotong Wang' ? (
                              <strong className="text-foreground">{author}</strong>
                            ) : (
                              author
                            )}
                          </span>
                        ))}
                      </p>
                      <p className="text-[11px] text-muted-foreground">{pub.venue}</p>
                      {pub.award && (
                        <span className="inline-flex items-center gap-1 text-[9px] font-bold uppercase tracking-wide text-primary bg-primary/10 px-1.5 py-0.5 rounded-sm">
                          🏆 {pub.award}
                        </span>
                      )}
                      <div className="flex flex-wrap gap-3 pt-0.5">
                        {pub.links.map((link) => (
                          <a
                            key={link.label}
                            href={link.url}
                            target="_blank"
                            rel="noopener noreferrer"
                            className="text-[10px] font-mono text-primary hover:opacity-70 transition-opacity"
                          >
                            [{link.label}]
                          </a>
                        ))}
                      </div>
                    </div>
                  </article>
                ))}
              </div>
            </section>
          </main>
        </div>

        {/* Footer */}
        <footer className="mt-16 pt-6 border-t border-border flex justify-between items-center">
          <p className="text-xs text-muted-foreground">
            © {new Date().getFullYear()} Ruotong Wang
          </p>
          <button
            onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
            className="p-2 rounded-full hover:bg-muted transition-colors text-muted-foreground"
            aria-label="Toggle theme"
          >
            {theme === 'dark' ? <Sun size={16} /> : <Moon size={16} />}
          </button>
        </footer>
      </div>
    </div>
  )
}
```

- [ ] **Step 3: Verify home page renders**

```bash
cd "Academic Personal Website"
npm run dev
```

Open `http://localhost:5173`. Verify:
- Profile photo loads in sidebar
- Bio text renders with correct links (Amy Zhang, Social Futures Lab, Haiyi Zhu, Macalester, GroupLens)
- Publications list appears with gradient thumbnails, author lists, venue labels, and links
- Writing sidebar section shows "All writing →" link (post list is empty until placeholder.mdx is added in Task 11 — that is expected)
- Dark/light toggle works in footer and persists after page reload
- Resize browser to < 1024px: sidebar collapses above the main content in a single column

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/pages/Home.tsx"
git commit -m "feat: implement Home page with sidebar, bio, and publications"
```

---

## Chunk 3: Blog — Writing, Post, MDX, Vercel

> **Prerequisites:** Tasks 7 (`usePosts.ts`) and 10 (`Home.tsx`) from Chunk 2 must be complete before this chunk.

### Task 9: Create usePosts shared hook

**Files:**
- Create: `Academic Personal Website/src/app/hooks/usePosts.ts`

- [ ] **Step 1: Create `src/app/hooks/usePosts.ts`**

```ts
export interface PostMeta {
  slug: string
  title: string
  date: string
  description: string
}

/**
 * Returns all blog posts sorted by date descending (newest first).
 * Reads from src/content/posts/*.mdx via import.meta.glob.
 */
export function getPosts(): PostMeta[] {
  const modules = import.meta.glob('../../content/posts/*.mdx', {
    eager: true,
  }) as Record<
    string,
    { frontmatter: { title: string; date: string; description: string } }
  >

  return Object.entries(modules)
    .map(([filePath, mod]) => ({
      slug: filePath.split('/').pop()!.replace('.mdx', ''),
      title: mod.frontmatter.title,
      date: mod.frontmatter.date,
      description: mod.frontmatter.description,
    }))
    .sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime())
}
```

- [ ] **Step 2: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/hooks/usePosts.ts"
git commit -m "feat: add usePosts hook for MDX post discovery"
```

---

### Task 11: Create placeholder MDX post

**Files:**
- Create: `Academic Personal Website/src/content/posts/placeholder.mdx`

- [ ] **Step 1: Create `src/content/posts/` directory**

```bash
mkdir -p "Academic Personal Website/src/content/posts"
```

- [ ] **Step 2: Create `src/content/posts/placeholder.mdx`**

```mdx
---
title: "A Template for Blog Posts"
date: "2026-03-16"
description: "This post demonstrates all supported MDX formatting. Use it as a reference when porting posts from Notion."
---

# Getting Started

This is a placeholder post. When you export a post from Notion as Markdown, paste the content below your frontmatter block. The table of contents on the right tracks h1, h2, and h3 headings automatically.

Body text uses EB Garamond at 18px. Paragraphs have generous spacing and line height for comfortable reading.

## Writing in Notion

Export your Notion page via **Export → Markdown & CSV**, then:

1. Create a new file: `src/content/posts/your-post-title.mdx`
2. Add the frontmatter block at the top (see below)
3. Paste your Notion markdown below the frontmatter
4. Commit and push — Vercel deploys in ~30 seconds

### Frontmatter Format

Every post needs these three fields at the very top:

```
---
title: "Your Post Title"
date: "YYYY-MM-DD"
description: "One sentence summary shown in the writing index."
---
```

## Text Formatting

You can write **bold text** and *italic text* inline. Links look [like this](https://example.com).

> Pull quotes use a left border and italic styling — useful for highlighting a key idea or quote from your research.

## Code

Inline `code` renders in monospace with a subtle background. Longer code blocks:

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"
```

## Lists

Unordered:
- First item
- Second item
- Third item

Ordered:
1. Step one
2. Step two
3. Step three

---

That covers the main formatting options. Delete this post when you add your first real post.
```

- [ ] **Step 3: Verify MDX compiles**

```bash
cd "Academic Personal Website"
npm run dev
```

Navigate to `http://localhost:5173/writing/placeholder`. Expected: page loads without console errors (Writing and Post pages don't exist yet — this just confirms MDX parsing works; a blank page or 404 is fine at this stage).

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/content/"
git commit -m "feat: add placeholder MDX post with formatting reference"
```

---

### Task 12: Implement Writing.tsx

**Files:**
- Rewrite: `Academic Personal Website/src/app/pages/Writing.tsx`

- [ ] **Step 1: Read existing `Writing.tsx`**

- [ ] **Step 2: Replace with:**

```tsx
import { Link } from 'react-router'
import { ArrowLeft } from 'lucide-react'
import { getPosts, type PostMeta } from '../hooks/usePosts'
import { NoiseOverlay } from '../components/NoiseOverlay'

function formatDate(dateStr: string): string {
  const d = new Date(dateStr)
  return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
}

export function Writing() {
  const posts = getPosts()

  // Group posts by year, sorted newest year first
  const byYear = posts.reduce<Record<string, PostMeta[]>>((acc, post) => {
    const year = new Date(post.date).getFullYear().toString()
    if (!acc[year]) acc[year] = []
    acc[year].push(post)
    return acc
  }, {})

  const years = Object.keys(byYear).sort((a, b) => parseInt(b) - parseInt(a))

  return (
    <div className="blog-page min-h-screen bg-background text-foreground transition-colors duration-500 selection:bg-primary/20 dark:selection:bg-primary/30">
      <NoiseOverlay />

      <div className="max-w-[860px] mx-auto px-8 py-12 relative z-10">

        {/* Back breadcrumb */}
        <Link
          to="/"
          className="blog-ui inline-flex items-center gap-2 text-sm text-muted-foreground hover:text-foreground transition-colors mb-12 group"
        >
          <ArrowLeft size={14} className="group-hover:-translate-x-0.5 transition-transform" />
          Back
        </Link>

        {/* Header */}
        <header className="mb-16">
          <h1 className="text-3xl font-bold text-foreground mb-2">Writing</h1>
          <p className="text-lg italic text-muted-foreground">
            Notes, essays, and working papers.
          </p>
        </header>

        {/* Post list grouped by year */}
        {posts.length === 0 ? (
          <p className="text-muted-foreground italic">No posts yet.</p>
        ) : (
          <div className="space-y-16">
            {years.map((year) => (
              <section
                key={year}
                className="grid grid-cols-1 md:grid-cols-[60px_1fr] gap-8"
              >
                <h2 className="blog-ui text-sm font-bold text-muted-foreground md:text-right sticky top-8 self-start">
                  {year}
                </h2>
                <div className="space-y-6">
                  {byYear[year].map((post) => (
                    <article key={post.slug} className="group">
                      <Link
                        to={`/writing/${post.slug}`}
                        className="flex flex-col sm:flex-row sm:items-baseline gap-2 sm:gap-6"
                      >
                        <span className="blog-ui text-xs font-bold text-muted-foreground w-20 shrink-0">
                          {formatDate(post.date)}
                        </span>
                        <h3 className="text-lg font-bold text-foreground group-hover:text-primary transition-colors underline decoration-border underline-offset-4 group-hover:decoration-primary/50">
                          {post.title}
                        </h3>
                      </Link>
                    </article>
                  ))}
                </div>
              </section>
            ))}
          </div>
        )}
      </div>
    </div>
  )
}
```

- [ ] **Step 3: Verify Writing page**

With dev server running, navigate to `http://localhost:5173/writing`.
Expected: "Writing" heading renders in EB Garamond, placeholder post appears under the correct year, "Back" breadcrumb links home.

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/pages/Writing.tsx"
git commit -m "feat: implement Writing index page with MDX post list"
```

---

### Task 13: Implement Post.tsx with MDX renderer and TOC

**Files:**
- Rewrite: `Academic Personal Website/src/app/pages/Post.tsx`

- [ ] **Step 1: Read existing `Post.tsx`**

- [ ] **Step 2: Replace with:**

```tsx
import { useParams, Link } from 'react-router'
import { useState, useEffect, useRef, type ComponentType } from 'react'
import { ArrowLeft } from 'lucide-react'
import { NoiseOverlay } from '../components/NoiseOverlay'

interface HeadingItem {
  id: string
  text: string
  level: 1 | 2 | 3
}

function formatDate(dateStr: string): string {
  return new Date(dateStr).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  })
}

export function Post() {
  const { slug } = useParams<{ slug: string }>()
  const [PostContent, setPostContent] = useState<ComponentType | null>(null)
  const [meta, setMeta] = useState<{ title: string; date: string } | null>(null)
  const [notFound, setNotFound] = useState(false)
  const [headings, setHeadings] = useState<HeadingItem[]>([])
  const [activeId, setActiveId] = useState('')
  const contentRef = useRef<HTMLDivElement>(null)

  // Load the MDX module that matches the slug
  useEffect(() => {
    setPostContent(null)
    setMeta(null)
    setNotFound(false)
    setHeadings([])

    const modules = import.meta.glob('../../content/posts/*.mdx')
    const key = Object.keys(modules).find((k) => k.endsWith(`/${slug}.mdx`))

    if (!key) {
      setNotFound(true)
      return
    }

    modules[key]().then((mod: any) => {
      setPostContent(() => mod.default)
      setMeta({
        title: mod.frontmatter?.title ?? slug ?? '',
        date: mod.frontmatter?.date ?? '',
      })
    })
  }, [slug])

  // Build TOC from rendered h1/h2/h3 elements, set up IntersectionObserver
  useEffect(() => {
    if (!contentRef.current || !PostContent) return

    const els = Array.from(
      contentRef.current.querySelectorAll('h1, h2, h3')
    ) as HTMLElement[]

    const items: HeadingItem[] = els.map((el) => {
      const text = el.textContent ?? ''
      const id =
        el.id ||
        text
          .toLowerCase()
          .replace(/[^\w\s-]/g, '')
          .trim()
          .replace(/\s+/g, '-')
      el.id = id
      return { id, text, level: parseInt(el.tagName[1]) as 1 | 2 | 3 }
    })

    setHeadings(items)

    const observer = new IntersectionObserver(
      (entries) => {
        const visible = entries.filter((e) => e.isIntersecting)
        if (visible.length > 0) {
          setActiveId(visible[0].target.id)
        }
      },
      { rootMargin: '0px 0px -70% 0px' }
    )

    els.forEach((el) => observer.observe(el))
    return () => els.forEach((el) => observer.unobserve(el))
  }, [PostContent])

  if (notFound) {
    return (
      <div className="blog-page min-h-screen bg-background text-foreground flex flex-col items-center justify-center gap-4">
        <p className="text-muted-foreground">Post not found.</p>
        <Link to="/writing" className="text-primary hover:underline underline-offset-2 text-sm">
          ← Back to Writing
        </Link>
      </div>
    )
  }

  if (!PostContent || !meta) {
    return (
      <div className="blog-page min-h-screen bg-background text-foreground" />
    )
  }

  return (
    <div className="blog-page min-h-screen bg-background text-foreground transition-colors duration-500 selection:bg-primary/20 dark:selection:bg-primary/30">
      <NoiseOverlay />

      <div className="max-w-[960px] mx-auto px-8 py-12 relative z-10">

        {/* Back breadcrumb */}
        <Link
          to="/writing"
          className="blog-ui inline-flex items-center gap-2 text-sm text-muted-foreground hover:text-foreground transition-colors mb-8 group"
        >
          <ArrowLeft size={14} className="group-hover:-translate-x-0.5 transition-transform" />
          Back
        </Link>

        {/* Post header */}
        <div className="mb-10 lg:max-w-[680px]">
          <h1 className="text-4xl font-bold text-foreground mb-3 leading-tight">
            {meta.title}
          </h1>
          {meta.date && (
            <p className="blog-ui text-xs font-bold uppercase tracking-widest text-muted-foreground">
              {formatDate(meta.date)}
            </p>
          )}
        </div>

        {/* Content + TOC */}
        <div className="flex flex-col lg:flex-row gap-16 items-start">

          {/* Main MDX content */}
          <div ref={contentRef} className="prose-mdx flex-1 lg:max-w-[680px] w-full">
            <PostContent />
          </div>

          {/* Sticky TOC sidebar — desktop only */}
          {headings.length > 0 && (
            <aside className="hidden lg:block w-[200px] shrink-0 sticky top-8 self-start">
              <nav aria-label="Table of contents">
                <ul className="space-y-2">
                  {headings.map((h) => (
                    <li
                      key={h.id}
                      style={{
                        paddingLeft:
                          h.level === 1 ? 0 : h.level === 2 ? '12px' : '24px',
                      }}
                    >
                      <a
                        href={`#${h.id}`}
                        className={`blog-ui block text-[13px] transition-colors leading-snug ${
                          activeId === h.id
                            ? 'text-foreground font-bold'
                            : 'text-muted-foreground hover:text-foreground'
                        }`}
                      >
                        {h.text}
                      </a>
                    </li>
                  ))}
                </ul>
              </nav>
            </aside>
          )}
        </div>
      </div>
    </div>
  )
}
```

- [ ] **Step 3: Verify Post page**

With dev server running, navigate to `http://localhost:5173/writing/placeholder`.
Expected:
- Post title "A Template for Blog Posts" renders in **EB Garamond** bold (inherited via `.blog-page`; headings h2/h3 in body use DM Sans)
- Body text renders in EB Garamond
- TOC sidebar appears on the right with headings from the post
- Scrolling highlights the active heading in the TOC
- "Back" breadcrumb links to `/writing`
- Navigating to `/writing/nonexistent` shows "Post not found."

- [ ] **Step 4: Commit**

```bash
cd ..
git add "Academic Personal Website/src/app/pages/Post.tsx"
git commit -m "feat: implement Post page with MDX renderer and h1/h2/h3 TOC"
```

---

### Task 14: Configure Vercel deployment

**Files:**
- Create: `Academic Personal Website/vercel.json`

- [ ] **Step 1: Create `vercel.json`**

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

This ensures React Router's client-side routing works correctly — all paths serve `index.html` and React Router handles the route.

- [ ] **Step 2: Verify build succeeds**

```bash
cd "Academic Personal Website"
npm run build
```

Expected: `dist/` directory created with no TypeScript or build errors. Any warnings about unused imports are acceptable; errors are not.

- [ ] **Step 3: Commit**

```bash
cd ..
git add "Academic Personal Website/vercel.json"
git commit -m "feat: add Vercel SPA routing config"
```

- [ ] **Step 4: Connect repo to Vercel**

1. Go to [vercel.com](https://vercel.com) → New Project → Import Git Repository
2. Select the `personal-website` repository
3. Set **Root Directory** to `Academic Personal Website`
4. Framework preset: **Vite** (auto-detected)
5. Build command: `npm run build` (default)
6. Output directory: `dist` (default)
7. Click **Deploy**

Expected: Vercel build completes, site is live at a `*.vercel.app` URL.

- [ ] **Step 5: Verify live site**

Open the Vercel URL. Verify:
- Home page loads with profile photo, bio, publications
- `/writing` renders the post list
- `/writing/placeholder` renders the placeholder post with TOC
- Dark mode toggle works
- Refreshing `/writing` does not 404 (the rewrite rule is working)

- [ ] **Step 6: Final commit**

```bash
git add .
git commit -m "chore: finalize website rebuild"
git push origin main
```

Expected: Vercel auto-deploys the pushed changes.

---

## Done

The rebuilt site is live. To publish a new blog post from Notion:

1. Export the Notion page as **Markdown & CSV**
2. Create `Academic Personal Website/src/content/posts/your-title.mdx`
3. Add frontmatter at the top:
   ```
   ---
   title: "Post Title"
   date: "YYYY-MM-DD"
   description: "One sentence summary."
   ---
   ```
4. Paste the Notion markdown below the frontmatter
5. `git add` → `git commit` → `git push` — Vercel deploys in ~30 seconds
