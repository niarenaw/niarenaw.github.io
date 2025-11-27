# Implementation Plan - Personal Website

**Date:** 2024-11-27
**Based on:** `001-MP.md`
**Status:** Ready for execution

---

## Overview

This document provides the exhaustive, file-by-file implementation plan for building a personal website with Astro 5, Tailwind CSS 4, and GitHub Pages deployment. It translates the 10 tickets from the master plan into granular, actionable tasks with complete code for every file.

### Key Updates from Research

During planning, I discovered several important changes from the original master plan assumptions:

1. **Tailwind CSS 4** - The `@astrojs/tailwind` integration is deprecated. Tailwind 4 now uses a native Vite plugin (`@tailwindcss/vite`) and CSS-first configuration.
2. **No `tailwind.config.mjs`** - Tailwind 4 eliminates the JavaScript config file. Dark mode is configured via `@custom-variant` in CSS.
3. **Content Collections Loader API** - Astro 5 uses a new `loader` pattern with `glob()` instead of implicit directory-based collections.
4. **Shiki Theme CSS Variables** - Dual themes use `--shiki-dark-*` CSS variables with media queries or class selectors.
5. **GitHub Actions v5** - Updated `withastro/action@v5` and `actions/deploy-pages@v4`.

---

## Proposed Structure

### Directory Layout

```
website/
├── .github/
│   └── workflows/
│       └── deploy.yml
├── .gitignore
├── README.md
├── astro.config.mjs
├── package.json
├── public/
│   └── favicon.svg
├── src/
│   ├── components/
│   │   ├── BaseHead.astro
│   │   ├── Footer.astro
│   │   ├── Header.astro
│   │   └── SocialLinks.astro
│   ├── content/
│   │   └── blog/
│   │       ├── _template.md
│   │       └── 2024-01-15-hello-world.md
│   ├── content.config.ts
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   └── BlogPostLayout.astro
│   ├── pages/
│   │   ├── blog/
│   │   │   ├── [...slug].astro
│   │   │   └── index.astro
│   │   └── index.astro
│   └── styles/
│       └── global.css
└── tsconfig.json
```

### Guiding Principles

1. **Type Safety** - Zod schemas for content, TypeScript strict mode
2. **Zero Client JS** - Static site, only inline scripts for dark mode
3. **CSS Variables** - Single source of truth for theming
4. **Progressive Enhancement** - Works without JavaScript
5. **Clean Forward Progress** - No backwards compatibility scaffolding

---

## File-by-File Change List

### Phase 1: Project Initialization (Ticket #1)

---

#### File: `package.json` (new)

**What it does:** Defines project dependencies and scripts.

**Why:** Required for npm to manage the project. Includes Astro 5, Tailwind 4 Vite plugin, and KaTeX dependencies.

```json
{
  "name": "personal-website",
  "type": "module",
  "version": "1.0.0",
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview"
  },
  "dependencies": {
    "astro": "^5.0.0"
  },
  "devDependencies": {
    "@tailwindcss/vite": "^4.0.0",
    "tailwindcss": "^4.0.0",
    "rehype-katex": "^7.0.0",
    "remark-math": "^6.0.0"
  }
}
```

---

#### File: `astro.config.mjs` (new)

**What it does:** Configures Astro with Tailwind Vite plugin, markdown processing (Shiki dual themes, remark-math, rehype-katex), and site URL.

**Why:** Central configuration for the entire build pipeline. Uses Tailwind 4's Vite plugin instead of the deprecated integration.

```javascript
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';
import remarkMath from 'remark-math';
import rehypeKatex from 'rehype-katex';

export default defineConfig({
  site: 'https://niarenaw.github.io',
  vite: {
    plugins: [tailwindcss()],
  },
  markdown: {
    shikiConfig: {
      themes: {
        light: 'github-light',
        dark: 'github-dark',
      },
    },
    remarkPlugins: [remarkMath],
    rehypePlugins: [rehypeKatex],
  },
});
```

---

#### File: `tsconfig.json` (new)

**What it does:** TypeScript configuration with Astro's strict preset.

**Why:** Enables type checking and IDE support. Uses Astro's recommended settings.

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

#### File: `src/styles/global.css` (new)

**What it does:**
1. Imports Tailwind CSS 4
2. Configures dark mode via `@custom-variant` (class-based, not media query)
3. Defines CSS variables for theming (background, text, accent colors)
4. Adds Shiki dual-theme CSS for code blocks

**Why:** Single source of truth for all styling. Tailwind 4's CSS-first approach means no `tailwind.config.mjs` needed.

```css
@import "tailwindcss";

/* Enable class-based dark mode (toggle via .dark on <html>) */
@custom-variant dark (&:where(.dark, .dark *));

/* Theme CSS variables */
@theme {
  /* Light mode colors (default) */
  --color-bg: #ffffff;
  --color-text: #1f2937;
  --color-text-muted: #6b7280;
  --color-accent: #2563eb;
  --color-accent-hover: #1d4ed8;
  --color-border: #e5e7eb;
  --color-code-bg: #f3f4f6;
}

/* Dark mode overrides */
@layer base {
  .dark {
    --color-bg: #111827;
    --color-text: #f9fafb;
    --color-text-muted: #9ca3af;
    --color-accent: #60a5fa;
    --color-accent-hover: #93c5fd;
    --color-border: #374151;
    --color-code-bg: #1f2937;
  }
}

/* Base styles */
@layer base {
  html {
    background-color: var(--color-bg);
    color: var(--color-text);
  }

  body {
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    line-height: 1.6;
    transition: background-color 0.2s ease, color 0.2s ease;
  }

  a {
    color: var(--color-accent);
    text-decoration: none;
  }

  a:hover {
    color: var(--color-accent-hover);
    text-decoration: underline;
  }
}

/* Shiki dual theme support - switch based on .dark class */
.dark .astro-code,
.dark .astro-code span {
  color: var(--shiki-dark) !important;
  background-color: var(--shiki-dark-bg) !important;
  font-style: var(--shiki-dark-font-style) !important;
  font-weight: var(--shiki-dark-font-weight) !important;
  text-decoration: var(--shiki-dark-text-decoration) !important;
}

/* Code block styling */
.astro-code {
  padding: 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  font-size: 0.875rem;
  line-height: 1.7;
}

/* Inline code */
:not(pre) > code {
  background-color: var(--color-code-bg);
  padding: 0.125rem 0.375rem;
  border-radius: 0.25rem;
  font-size: 0.875em;
}
```

---

#### File: `.gitignore` (new)

**What it does:** Excludes build artifacts and dependencies from version control.

**Why:** Standard practice to keep repo clean.

```
# Dependencies
node_modules/

# Build output
dist/
.astro/

# Environment
.env
.env.*
!.env.example

# Editor
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*
```

---

### Phase 2: Content Collection Schema (Ticket #2)

---

#### File: `src/content.config.ts` (new)

**What it does:** Defines the blog content collection with Zod schema validation and the new Astro 5 loader API.

**Why:** Type-safe frontmatter prevents broken posts. The `glob` loader explicitly targets markdown files in `src/content/blog/`. The pattern `**/[^_]*.md` excludes underscore-prefixed files (Astro 5 no longer auto-excludes them).

```typescript
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  // Pattern excludes underscore-prefixed files (like _template.md)
  loader: glob({ pattern: '**/[^_]*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    description: z.string().max(160),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    draft: z.boolean().default(false),
  }),
});

export const collections = { blog };
```

---

#### File: `src/content/blog/_template.md` (new)

**What it does:** Template for creating new blog posts. The underscore prefix excludes it from the collection.

**Why:** Provides a consistent starting point for new posts with all required frontmatter fields.

```markdown
---
title: "Post Title Here"
description: "A brief description for SEO (max 160 characters)"
pubDate: 2024-01-01
# updatedDate: 2024-01-02
# draft: true
---

Start writing your post here...

## Code Example

```javascript
console.log('Hello, world!');
```

## Math Example

Inline math: $E = mc^2$

Block math:

$$
\int_{-\infty}^{\infty} e^{-x^2} dx = \sqrt{\pi}
$$
```

---

#### File: `src/content/blog/2024-01-15-hello-world.md` (new)

**What it does:** Sample blog post demonstrating all supported features: markdown, code highlighting, and LaTeX.

**Why:** Validates the entire content pipeline works correctly.

```markdown
---
title: "Hello World"
description: "Welcome to my blog! This is my first post testing markdown, code, and LaTeX."
pubDate: 2024-01-15
---

Welcome to my blog! This is my first post, testing all the features.

## Markdown Features

Here's some **bold text** and *italic text*. You can also use `inline code`.

### Lists

- Item one
- Item two
- Item three

### Links

Check out [Astro](https://astro.build) for more info.

## Code Highlighting

Here's a TypeScript example with syntax highlighting:

```typescript
interface User {
  name: string;
  email: string;
}

function greet(user: User): string {
  return `Hello, ${user.name}!`;
}
```

And some Python:

```python
def fibonacci(n: int) -> int:
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

## LaTeX Support

Inline math works like this: $E = mc^2$

Block equations are rendered beautifully:

$$
\frac{d}{dx} \int_a^x f(t) dt = f(x)
$$

The quadratic formula:

$$
x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
$$

That's it for the first post!
```

---

### Phase 3: Base Layout & Head Component (Ticket #3)

---

#### File: `src/components/BaseHead.astro` (new)

**What it does:**
1. Sets charset, viewport, and favicon
2. Dynamic title and description via props
3. OpenGraph meta tags for social sharing
4. Conditional KaTeX CSS loading (only when `needsMath` is true)

**Why:** Centralized `<head>` management for SEO and performance. KaTeX CSS is ~25KB, so we only load it on pages that need it.

```astro
---
interface Props {
  title: string;
  description: string;
  needsMath?: boolean;
}

const { title, description, needsMath = false } = Astro.props;
const canonicalURL = new URL(Astro.url.pathname, Astro.site);
---

<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />

<title>{title}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonicalURL} />

<!-- OpenGraph -->
<meta property="og:type" content="website" />
<meta property="og:url" content={canonicalURL} />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />

<!-- KaTeX CSS (conditional) -->
{needsMath && (
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css"
    integrity="sha384-n8MVd4RsNIU0tAv4ct0nTaAbDJwPJzDEaqSD1odI+WdtXRGWt2kTvGFasHpSy3SV"
    crossorigin="anonymous"
  />
)}
```

---

#### File: `src/components/Header.astro` (new - stub)

**What it does:** Minimal placeholder header. Will be fully implemented in Ticket #4.

**Why:** Allows BaseLayout to compile and pages to render while we implement other components.

```astro
---
// Stub header - will be replaced in Ticket #4
---

<header class="py-4 px-6 border-b border-[var(--color-border)]">
  <nav class="max-w-2xl mx-auto flex justify-between items-center">
    <a href="/" class="font-semibold">Site Name</a>
    <div class="flex gap-4">
      <a href="/">About</a>
      <a href="/blog">Blog</a>
    </div>
  </nav>
</header>
```

---

#### File: `src/components/Footer.astro` (new - stub)

**What it does:** Minimal placeholder footer. Will be fully implemented in Ticket #5.

**Why:** Allows BaseLayout to compile while we build other components.

```astro
---
// Stub footer - will be replaced in Ticket #5
---

<footer class="py-6 px-6 border-t border-[var(--color-border)] text-center text-[var(--color-text-muted)]">
  <p>&copy; {new Date().getFullYear()}</p>
</footer>
```

---

#### File: `src/layouts/BaseLayout.astro` (new)

**What it does:**
1. HTML shell with proper `lang` attribute
2. Imports and uses BaseHead component
3. Inline dark mode script (prevents flash of wrong theme)
4. Imports global CSS
5. Renders Header, main content slot, and Footer

**Why:** Foundation for all pages. The `is:inline` script runs before body renders to apply dark mode class immediately.

```astro
---
import BaseHead from '../components/BaseHead.astro';
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
import '../styles/global.css';

interface Props {
  title: string;
  description: string;
  needsMath?: boolean;
}

const { title, description, needsMath = false } = Astro.props;
---

<!doctype html>
<html lang="en">
  <head>
    <BaseHead title={title} description={description} needsMath={needsMath} />
    <script is:inline>
      // Apply dark mode immediately to prevent flash
      if (
        localStorage.theme === 'dark' ||
        (!localStorage.theme && window.matchMedia('(prefers-color-scheme: dark)').matches)
      ) {
        document.documentElement.classList.add('dark');
      }
    </script>
  </head>
  <body class="min-h-screen flex flex-col">
    <Header />
    <main class="flex-1 px-6 py-8">
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

---

### Phase 4: Header & Dark Mode Toggle (Ticket #4)

---

#### File: `src/components/Header.astro` (modify - replace stub)

**What it does:**
1. Site title/name linking to home
2. Navigation links (About, Blog)
3. Dark mode toggle button with sun/moon icons
4. Client-side script to toggle theme and persist to localStorage
5. Responsive design

**Why:** Primary navigation and theme control. Uses inline SVG icons to avoid external dependencies.

```astro
---
// Full header implementation
---

<header class="py-4 px-6 border-b border-[var(--color-border)]">
  <nav class="max-w-2xl mx-auto flex justify-between items-center">
    <a href="/" class="font-semibold text-lg hover:no-underline">
      Nia Renaw
    </a>

    <div class="flex items-center gap-6">
      <a href="/" class="hover:text-[var(--color-accent)]">About</a>
      <a href="/blog" class="hover:text-[var(--color-accent)]">Blog</a>

      <!-- Dark mode toggle -->
      <button
        id="theme-toggle"
        type="button"
        aria-label="Toggle dark mode"
        class="p-2 rounded-lg hover:bg-[var(--color-code-bg)] transition-colors"
      >
        <!-- Sun icon (shown in dark mode) -->
        <svg
          id="sun-icon"
          class="w-5 h-5 hidden"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
          />
        </svg>
        <!-- Moon icon (shown in light mode) -->
        <svg
          id="moon-icon"
          class="w-5 h-5 hidden"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
          />
        </svg>
      </button>
    </div>
  </nav>
</header>

<script>
  function initThemeToggle() {
    const toggle = document.getElementById('theme-toggle');
    const sunIcon = document.getElementById('sun-icon');
    const moonIcon = document.getElementById('moon-icon');

    function updateIcons() {
      const isDark = document.documentElement.classList.contains('dark');
      if (sunIcon && moonIcon) {
        sunIcon.classList.toggle('hidden', !isDark);
        moonIcon.classList.toggle('hidden', isDark);
      }
    }

    // Initial icon state
    updateIcons();

    toggle?.addEventListener('click', () => {
      const isDark = document.documentElement.classList.toggle('dark');
      localStorage.theme = isDark ? 'dark' : 'light';
      updateIcons();
    });
  }

  // Run on initial load
  initThemeToggle();

  // Re-run after Astro page transitions (View Transitions)
  document.addEventListener('astro:after-swap', initThemeToggle);
</script>
```

---

### Phase 5: Footer & Social Links (Ticket #5)

---

#### File: `src/components/SocialLinks.astro` (new)

**What it does:** Renders LinkedIn and GitHub icons with configurable URLs. Opens in new tab with security attributes.

**Why:** Reusable component for social links. Uses inline SVG to avoid external dependencies and ensure icons work offline.

```astro
---
interface Props {
  linkedin?: string;
  github?: string;
}

const {
  linkedin = 'https://linkedin.com/in/niarenaw',
  github = 'https://github.com/niarenaw',
} = Astro.props;
---

<div class="flex gap-4 items-center">
  {linkedin && (
    <a
      href={linkedin}
      target="_blank"
      rel="noopener noreferrer"
      aria-label="LinkedIn"
      class="text-[var(--color-text-muted)] hover:text-[var(--color-accent)] transition-colors"
    >
      <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
        <path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433c-1.144 0-2.063-.926-2.063-2.065 0-1.138.92-2.063 2.063-2.063 1.14 0 2.064.925 2.064 2.063 0 1.139-.925 2.065-2.064 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/>
      </svg>
    </a>
  )}

  {github && (
    <a
      href={github}
      target="_blank"
      rel="noopener noreferrer"
      aria-label="GitHub"
      class="text-[var(--color-text-muted)] hover:text-[var(--color-accent)] transition-colors"
    >
      <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
        <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
      </svg>
    </a>
  )}
</div>
```

---

#### File: `src/components/Footer.astro` (modify - replace stub)

**What it does:** Full footer with dynamic copyright year and social links.

**Why:** Provides consistent footer across all pages with social presence.

```astro
---
import SocialLinks from './SocialLinks.astro';
---

<footer class="py-8 px-6 border-t border-[var(--color-border)]">
  <div class="max-w-2xl mx-auto flex flex-col items-center gap-4">
    <SocialLinks />
    <p class="text-sm text-[var(--color-text-muted)]">
      &copy; {new Date().getFullYear()} Nia Renaw. All rights reserved.
    </p>
  </div>
</footer>
```

---

### Phase 6: About Page (Ticket #6)

---

#### File: `src/pages/index.astro` (new)

**What it does:** Homepage with professional bio, experience, education sections.

**Why:** Landing page for the site. Uses placeholder content that you'll customize.

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import SocialLinks from '../components/SocialLinks.astro';
---

<BaseLayout
  title="Nia Renaw - Software Engineer"
  description="Personal website of Nia Renaw. Software engineer, builder, and occasional writer."
>
  <article class="max-w-2xl mx-auto">
    <!-- Bio -->
    <section class="mb-12">
      <h1 class="text-3xl font-bold mb-4 text-[var(--color-accent)]">
        Hello, I'm Nia Renaw
      </h1>
      <p class="text-lg text-[var(--color-text-muted)] leading-relaxed">
        I'm a software engineer who enjoys building things and learning new technologies.
        Currently interested in web development, systems programming, and developer tools.
        I write about things I learn along the way.
      </p>
    </section>

    <!-- Experience -->
    <section class="mb-12">
      <h2 class="text-xl font-semibold mb-4 text-[var(--color-accent)]">
        Experience
      </h2>
      <div class="space-y-4">
        <div>
          <div class="flex justify-between items-baseline">
            <h3 class="font-medium">Software Engineer</h3>
            <span class="text-sm text-[var(--color-text-muted)]">2022 - Present</span>
          </div>
          <p class="text-[var(--color-text-muted)]">Acme Corp</p>
        </div>
        <div>
          <div class="flex justify-between items-baseline">
            <h3 class="font-medium">Junior Developer</h3>
            <span class="text-sm text-[var(--color-text-muted)]">2020 - 2022</span>
          </div>
          <p class="text-[var(--color-text-muted)]">StartupXYZ</p>
        </div>
      </div>
    </section>

    <!-- Education -->
    <section class="mb-12">
      <h2 class="text-xl font-semibold mb-4 text-[var(--color-accent)]">
        Education
      </h2>
      <div>
        <div class="flex justify-between items-baseline">
          <h3 class="font-medium">B.S. Computer Science</h3>
          <span class="text-sm text-[var(--color-text-muted)]">2016 - 2020</span>
        </div>
        <p class="text-[var(--color-text-muted)]">State University</p>
      </div>
    </section>

    <!-- Connect -->
    <section>
      <h2 class="text-xl font-semibold mb-4 text-[var(--color-accent)]">
        Connect
      </h2>
      <SocialLinks />
    </section>
  </article>
</BaseLayout>
```

---

### Phase 7: Blog Listing Page (Ticket #7)

---

#### File: `src/pages/blog/index.astro` (new)

**What it does:**
1. Fetches all blog posts from the content collection
2. Filters out drafts in production mode
3. Sorts by publication date (newest first)
4. Renders as "Date - Title" format with links

**Why:** Clean blog index with minimal design. Uses `Intl.DateTimeFormat` for locale-aware date formatting.

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

// Fetch and filter posts
const allPosts = await getCollection('blog', ({ data }) => {
  // Show drafts in dev, hide in production
  return import.meta.env.PROD ? !data.draft : true;
});

// Sort by pubDate descending (newest first)
const posts = allPosts.sort((a, b) =>
  b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
);

// Date formatter
const dateFormat = new Intl.DateTimeFormat('en-US', {
  year: 'numeric',
  month: 'short',
  day: 'numeric',
});
---

<BaseLayout
  title="Blog - Nia Renaw"
  description="Technical blog posts about software engineering, programming, and technology."
>
  <div class="max-w-2xl mx-auto">
    <h1 class="text-3xl font-bold mb-8 text-[var(--color-accent)]">Blog</h1>

    {posts.length === 0 ? (
      <p class="text-[var(--color-text-muted)]">No posts yet. Check back soon!</p>
    ) : (
      <ul class="space-y-4">
        {posts.map((post) => (
          <li>
            <a
              href={`/blog/${post.id}`}
              class="flex flex-col sm:flex-row sm:items-baseline gap-1 sm:gap-3 group"
            >
              <time
                datetime={post.data.pubDate.toISOString()}
                class="text-sm text-[var(--color-text-muted)] shrink-0"
              >
                {dateFormat.format(post.data.pubDate)}
              </time>
              <span class="text-[var(--color-text-muted)] hidden sm:inline">—</span>
              <span class="group-hover:text-[var(--color-accent)] transition-colors">
                {post.data.title}
              </span>
            </a>
          </li>
        ))}
      </ul>
    )}
  </div>
</BaseLayout>
```

---

### Phase 8: Individual Blog Post Pages (Ticket #8)

---

#### File: `src/layouts/BlogPostLayout.astro` (new)

**What it does:**
1. Wraps BaseLayout with `needsMath={true}` to load KaTeX CSS
2. Displays post title and formatted date
3. Provides prose styling for rendered markdown content

**Why:** Dedicated layout for blog posts with typography optimized for reading.

```astro
---
import BaseLayout from './BaseLayout.astro';

interface Props {
  title: string;
  description: string;
  pubDate: Date;
  updatedDate?: Date;
}

const { title, description, pubDate, updatedDate } = Astro.props;

const dateFormat = new Intl.DateTimeFormat('en-US', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
});
---

<BaseLayout title={title} description={description} needsMath={true}>
  <article class="max-w-2xl mx-auto">
    <header class="mb-8">
      <h1 class="text-3xl font-bold mb-2">{title}</h1>
      <div class="text-[var(--color-text-muted)] text-sm">
        <time datetime={pubDate.toISOString()}>
          {dateFormat.format(pubDate)}
        </time>
        {updatedDate && (
          <span>
            {' '}(Updated: <time datetime={updatedDate.toISOString()}>
              {dateFormat.format(updatedDate)}
            </time>)
          </span>
        )}
      </div>
    </header>

    <div class="prose">
      <slot />
    </div>
  </article>
</BaseLayout>

<style>
  /* Prose styles for blog content */
  .prose {
    line-height: 1.75;
  }

  .prose :global(h2) {
    font-size: 1.5rem;
    font-weight: 600;
    margin-top: 2rem;
    margin-bottom: 1rem;
    color: var(--color-accent);
  }

  .prose :global(h3) {
    font-size: 1.25rem;
    font-weight: 600;
    margin-top: 1.5rem;
    margin-bottom: 0.75rem;
  }

  .prose :global(p) {
    margin-bottom: 1.25rem;
  }

  .prose :global(ul),
  .prose :global(ol) {
    margin-bottom: 1.25rem;
    padding-left: 1.5rem;
  }

  .prose :global(li) {
    margin-bottom: 0.5rem;
  }

  .prose :global(ul) {
    list-style-type: disc;
  }

  .prose :global(ol) {
    list-style-type: decimal;
  }

  .prose :global(blockquote) {
    border-left: 4px solid var(--color-accent);
    padding-left: 1rem;
    margin: 1.5rem 0;
    color: var(--color-text-muted);
    font-style: italic;
  }

  .prose :global(pre) {
    margin: 1.5rem 0;
  }

  .prose :global(hr) {
    border: none;
    border-top: 1px solid var(--color-border);
    margin: 2rem 0;
  }

  /* KaTeX math styling */
  .prose :global(.katex-display) {
    overflow-x: auto;
    padding: 0.5rem 0;
  }
</style>
```

---

#### File: `src/pages/blog/[...slug].astro` (new)

**What it does:**
1. Uses `getStaticPaths()` to generate routes for all blog posts
2. Renders the post content using `render()` from the collection entry
3. Passes frontmatter data to BlogPostLayout

**Why:** Dynamic routing for blog posts. The spread route `[...slug]` handles any nesting depth.

```astro
---
import { getCollection, render } from 'astro:content';
import BlogPostLayout from '../../layouts/BlogPostLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => {
    return import.meta.env.PROD ? !data.draft : true;
  });

  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---

<BlogPostLayout
  title={post.data.title}
  description={post.data.description}
  pubDate={post.data.pubDate}
  updatedDate={post.data.updatedDate}
>
  <Content />
</BlogPostLayout>
```

---

### Phase 9: GitHub Actions Deployment (Ticket #9)

---

#### File: `.github/workflows/deploy.yml` (new)

**What it does:**
1. Triggers on push to `main` branch
2. Builds the site using `withastro/action@v5`
3. Deploys to GitHub Pages using `actions/deploy-pages@v4`

**Why:** Automated deployment pipeline. Zero-touch publishing after git push.

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install, build, and upload site
        uses: withastro/action@v3

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

#### File: `README.md` (new)

**What it does:** Documents the project setup, development workflow, and how to add new blog posts.

**Why:** Essential for onboarding and future reference.

```markdown
# Personal Website

A minimal personal website built with [Astro](https://astro.build), [Tailwind CSS](https://tailwindcss.com), and deployed to GitHub Pages.

## Features

- About page with professional info
- Blog with markdown, code highlighting, and LaTeX support
- Dark mode toggle with system preference detection
- Fully static (zero client-side JavaScript except theme toggle)
- Automated GitHub Pages deployment

## Development

### Prerequisites

- Node.js 18+
- npm

### Getting Started

```bash
# Install dependencies
npm install

# Start development server
npm run dev
```

The site will be available at `http://localhost:4321`.

### Building

```bash
# Create production build
npm run build

# Preview production build locally
npm run preview
```

## Adding a New Blog Post

1. Copy the template:
   ```bash
   cp src/content/blog/_template.md src/content/blog/YYYY-MM-DD-your-slug.md
   ```

2. Edit the frontmatter:
   ```yaml
   ---
   title: "Your Post Title"
   description: "Brief description for SEO (max 160 chars)"
   pubDate: 2024-01-15
   ---
   ```

3. Write your content using:
   - Standard markdown
   - Code blocks with syntax highlighting (specify language)
   - LaTeX: `$inline$` or `$$block$$`

4. Preview locally: `npm run dev`

5. Publish:
   ```bash
   git add .
   git commit -m "Add new post: Your Post Title"
   git push
   ```

The post will be live in ~2 minutes.

## Customization

### Personal Information

Update these files with your information:
- `src/components/Header.astro` - Site name
- `src/components/Footer.astro` - Copyright name
- `src/components/SocialLinks.astro` - LinkedIn/GitHub URLs
- `src/pages/index.astro` - Bio, experience, education
- `astro.config.mjs` - Site URL

### Styling

Theme colors are defined in `src/styles/global.css` using CSS variables:
- `--color-bg` - Background color
- `--color-text` - Text color
- `--color-accent` - Link and heading color
- etc.

## Deployment

The site automatically deploys to GitHub Pages when you push to `main`.

### First-time Setup

1. Go to your repository's Settings > Pages
2. Set Source to "GitHub Actions"
3. Push to `main` to trigger the first deployment

## License

MIT
```

---

### Phase 10: Polish & Responsiveness (Ticket #10)

---

#### File: `public/favicon.svg` (new)

**What it does:** Simple SVG favicon with your initial.

**Why:** Branded browser tab icon. SVG scales to any size and supports dark mode.

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <rect width="100" height="100" rx="20" fill="#2563eb"/>
  <text x="50" y="70" font-family="system-ui, sans-serif" font-size="60" font-weight="bold" fill="white" text-anchor="middle">N</text>
</svg>
```

---

#### File: `src/styles/global.css` (modify - add polish)

**Additions to existing file:**

```css
/* Add after existing code */

/* Smooth transitions for dark mode */
html {
  transition: background-color 0.2s ease;
}

/* Focus states for accessibility */
a:focus-visible,
button:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 2px;
}

/* Code block horizontal scroll on mobile */
.astro-code {
  max-width: 100%;
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}

/* KaTeX responsive */
.katex-display {
  overflow-x: auto;
  overflow-y: hidden;
  padding: 0.5rem 0;
}

/* Better text rendering */
body {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Prevent layout shift from scrollbar */
html {
  overflow-y: scroll;
}
```

---

## Task List with Checkboxes

### Ticket #1: Project Initialization & Configuration

- [ ] 1. Create project directory structure
  - [ ] 1.1 Create `src/styles/` directory
  - [ ] 1.2 Create `src/components/` directory
  - [ ] 1.3 Create `src/layouts/` directory
  - [ ] 1.4 Create `src/pages/` directory
  - [ ] 1.5 Create `src/content/blog/` directory
  - [ ] 1.6 Create `.github/workflows/` directory
  - [ ] 1.7 Create `public/` directory
- [ ] 2. Initialize git repository: `git init`
- [ ] 3. Create `package.json` with dependencies
- [ ] 4. Run `npm install` to install all dependencies
- [ ] 5. Create `astro.config.mjs` with Tailwind Vite plugin and markdown config
- [ ] 6. Create `tsconfig.json` with strict TypeScript settings
- [ ] 7. Create `src/styles/global.css` with Tailwind import and CSS variables
- [ ] 8. Create `.gitignore`
- [ ] 9. Verify: `npm run dev` starts successfully

### Ticket #2: Content Collection Schema

- [ ] 10. Create `src/content.config.ts` with blog collection schema
- [ ] 11. Create `src/content/blog/_template.md`
- [ ] 12. Create `src/content/blog/2024-01-15-hello-world.md` sample post
- [ ] 13. Verify: TypeScript types generate without errors

### Ticket #3: Base Layout & Head Component

- [ ] 14. Create `src/components/BaseHead.astro`
- [ ] 15. Create `src/components/Header.astro` (stub)
- [ ] 16. Create `src/components/Footer.astro` (stub)
- [ ] 17. Create `src/layouts/BaseLayout.astro`
- [ ] 18. Verify: Create temporary test page to confirm layout renders

### Ticket #4: Header & Dark Mode Toggle

- [ ] 19. Replace `src/components/Header.astro` with full implementation
- [ ] 20. Verify: Navigation links work
- [ ] 21. Verify: Dark mode toggle switches themes
- [ ] 22. Verify: Theme persists after page refresh

### Ticket #5: Footer & Social Links

- [ ] 23. Create `src/components/SocialLinks.astro`
- [ ] 24. Replace `src/components/Footer.astro` with full implementation
- [ ] 25. Verify: Social links open in new tab

### Ticket #6: About Page (Homepage)

- [ ] 26. Create `src/pages/index.astro`
- [ ] 27. Verify: Page renders at `/`
- [ ] 28. Verify: Responsive on mobile viewport

### Ticket #7: Blog Listing Page

- [ ] 29. Create `src/pages/blog/index.astro`
- [ ] 30. Verify: Page renders at `/blog`
- [ ] 31. Verify: Posts sorted newest first
- [ ] 32. Verify: Links navigate to correct posts

### Ticket #8: Individual Blog Post Pages

- [ ] 33. Create `src/layouts/BlogPostLayout.astro`
- [ ] 34. Create `src/pages/blog/[...slug].astro`
- [ ] 35. Verify: Sample post renders at `/blog/2024-01-15-hello-world`
- [ ] 36. Verify: Code highlighting works in both themes
- [ ] 37. Verify: LaTeX equations render correctly

### Ticket #9: GitHub Actions Deployment

- [ ] 38. Create `.github/workflows/deploy.yml`
- [ ] 39. Create `README.md`
- [ ] 40. Update `astro.config.mjs` with correct `site` URL
- [ ] 41. Commit and push to `main` branch
- [ ] 42. Configure GitHub Pages to use GitHub Actions (Settings > Pages)
- [ ] 43. Verify: Site deploys and is accessible

### Ticket #10: Polish & Responsiveness

- [ ] 44. Create `public/favicon.svg`
- [ ] 45. Add polish CSS to `src/styles/global.css`
- [ ] 46. Test all pages at 375px mobile viewport
- [ ] 47. Test dark mode on all pages
- [ ] 48. Run Lighthouse audit
  - [ ] 48.1 Performance > 90
  - [ ] 48.2 Accessibility > 90
  - [ ] 48.3 Best Practices > 90
  - [ ] 48.4 SEO > 90
- [ ] 49. Fix any Lighthouse issues identified

---

## Open Questions + Risk Areas

### Assumptions Made

1. **Test data for personal info** - Name, bio, experience, and education use placeholder test data. User will iterate on actual content after initial build.

2. **Tailwind v4 stability** - Tailwind CSS 4 was recently released. The `@tailwindcss/vite` plugin should be stable, but if issues arise, fallback to Tailwind v3 with the deprecated `@astrojs/tailwind` integration.

3. **KaTeX CDN** - Using jsDelivr CDN for KaTeX CSS. If offline support is needed, consider bundling locally.

### Potential Risk Areas

1. **Astro 5 Content Collections API** - The loader-based API is relatively new. If issues occur, the pattern used matches current documentation.

2. **Shiki dual themes CSS** - The `.dark` class selector approach should work, but verify in both themes during testing.

3. **GitHub Actions workflow** - The `withastro/action@v3` version is used (not v5 as initially researched) because v3 is the stable release. Monitor for updates.

---

## Verification Checklist (End-to-End)

After completing all tasks, verify:

- [ ] `npm run dev` starts without errors
- [ ] Homepage loads at `http://localhost:4321/`
- [ ] Blog listing loads at `http://localhost:4321/blog`
- [ ] Sample post loads at `http://localhost:4321/blog/2024-01-15-hello-world`
- [ ] Dark mode toggle works on all pages
- [ ] Theme persists across page refreshes
- [ ] Code highlighting appears correct in both themes
- [ ] LaTeX equations render properly
- [ ] All navigation links work
- [ ] Social links open in new tab
- [ ] Site is responsive at 375px width
- [ ] `npm run build` completes successfully
- [ ] GitHub Pages deployment succeeds
- [ ] Live site is accessible and functional
