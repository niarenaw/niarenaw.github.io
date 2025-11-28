# Codebase Documentation

> **Purpose**: This document serves as the authoritative reference for understanding and modifying this codebase. Read this before making UI changes, adding blog posts, or implementing new features.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [File Structure](#file-structure)
4. [Architecture](#architecture)
5. [UI System & Design](#ui-system--design)
6. [Content Management](#content-management)
7. [Components Reference](#components-reference)
8. [Configuration Files](#configuration-files)
9. [Best Practices](#best-practices)
10. [Common Tasks](#common-tasks)
11. [Troubleshooting](#troubleshooting)

---

## Project Overview

This is a **personal website and technical blog** for Nicholas Arena, built as a static site. The site prioritizes:

- **Performance**: Zero client-side JavaScript by default (except dark mode toggle)
- **Simplicity**: Markdown-based content, no CMS
- **Accessibility**: Semantic HTML, keyboard navigation, high contrast
- **Developer Experience**: Type-safe content with Zod validation

**Live URL**: `https://niarenaw.github.io`

### What This Site Contains

| Section | Route | Description |
|---------|-------|-------------|
| About/Home | `/` | Bio, professional experience, education |
| Blog Index | `/blog` | List of all published blog posts |
| Blog Posts | `/blog/[slug]` | Individual technical articles |

---

## Technology Stack

### Core Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| **Astro** | 5.x | Static site generator with zero JS by default |
| **Tailwind CSS** | 4.0 | Utility-first CSS framework |
| **TypeScript** | Strict | Type safety throughout the codebase |

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `@tailwindcss/vite` | Tailwind 4's Vite-first integration |
| `remark-math` | Parse LaTeX in markdown |
| `rehype-katex` | Render LaTeX to HTML |
| `shiki` | Syntax highlighting (built into Astro) |

### Important Version Notes

- **Tailwind 4.0** uses CSS-first configuration, NOT the deprecated `@astrojs/tailwind` integration
- **Astro 5** uses the new `glob()` loader for content collections, NOT the legacy `defineCollection` pattern
- **KaTeX** CSS is loaded from jsDelivr CDN, version `0.16.9`

---

## File Structure

```
website/
├── .docs/                    # Documentation (this file)
├── .github/                  # GitHub Actions workflows
├── .plans/                   # Planning documents (historical)
├── public/                   # Static assets (favicon, images)
├── src/
│   ├── components/           # Reusable Astro components
│   │   ├── BaseHead.astro    # SEO meta tags, conditional CSS
│   │   ├── Header.astro      # Navigation + dark mode toggle
│   │   ├── Footer.astro      # Copyright + social links
│   │   └── SocialLinks.astro # LinkedIn/GitHub icons
│   ├── content/              # Markdown content
│   │   └── blog/             # Blog posts directory
│   │       ├── _template.md  # Template for new posts
│   │       └── *.md          # Published posts
│   ├── layouts/              # Page layout templates
│   │   ├── BaseLayout.astro  # Root HTML shell
│   │   └── BlogPostLayout.astro  # Blog post wrapper
│   ├── pages/                # Route definitions (file-based routing)
│   │   ├── index.astro       # Home/About page
│   │   └── blog/
│   │       ├── index.astro   # Blog listing
│   │       └── [...slug].astro  # Dynamic blog post routes
│   ├── styles/
│   │   └── global.css        # Tailwind + theme CSS variables
│   └── content.config.ts     # Content collection schema
├── astro.config.mjs          # Astro build configuration
├── package.json              # Dependencies and scripts
└── tsconfig.json             # TypeScript configuration
```

### Key Directories Explained

| Directory | Do NOT... | Do... |
|-----------|-----------|-------|
| `src/components/` | Create page-specific components here | Keep components reusable across pages |
| `src/content/blog/` | Put non-markdown files here | Follow `YYYY-MM-DD-slug.md` naming |
| `src/layouts/` | Add business logic here | Keep layouts focused on structure |
| `src/pages/` | Put reusable UI here | Create routes and fetch data here |
| `public/` | Put CSS/JS here | Put static assets (images, favicon) |

---

## Architecture

### Build-Time Data Flow

```
┌─────────────────┐
│  Markdown Files │  src/content/blog/*.md
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Content Loader  │  glob() pattern matching
│ + Zod Schema    │  Frontmatter validation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Markdown Plugins│  remark-math → rehype-katex
│ + Shiki         │  Syntax highlighting
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Astro Compiler  │  Generate static HTML
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  /dist/ folder  │  Ready for deployment
└─────────────────┘
```

### Component Hierarchy

```
BaseLayout.astro
├── BaseHead.astro (in <head>)
│   └── Conditional KaTeX CSS
├── Header.astro
│   ├── Navigation links
│   └── Dark mode toggle (client JS)
├── <slot /> (page content)
└── Footer.astro
    └── SocialLinks.astro
```

### Routing System

Astro uses **file-based routing**:

| File | Route | Notes |
|------|-------|-------|
| `pages/index.astro` | `/` | Static page |
| `pages/blog/index.astro` | `/blog` | Lists all posts |
| `pages/blog/[...slug].astro` | `/blog/*` | Dynamic routes from content collection |

The `[...slug].astro` file uses `getStaticPaths()` to generate routes at build time from the blog content collection.

---

## UI System & Design

### Design Philosophy

1. **Minimalist**: Clean whitespace, focused content
2. **Readable**: Optimized typography for long-form reading
3. **Accessible**: WCAG compliant colors, semantic HTML
4. **Fast**: No JavaScript bloat, conditional asset loading

### Color System

All colors are defined as CSS custom properties in `src/styles/global.css`:

#### Light Mode (Default)

| Variable | Value | Usage |
|----------|-------|-------|
| `--color-bg` | `#ffffff` | Page background |
| `--color-text` | `#1f2937` | Primary text (gray-800) |
| `--color-text-muted` | `#6b7280` | Secondary text (gray-500) |
| `--color-accent` | `#2563eb` | Links, buttons (blue-600) |
| `--color-accent-hover` | `#1d4ed8` | Link hover state (blue-700) |
| `--color-border` | `#e5e7eb` | Borders, dividers (gray-200) |
| `--color-code-bg` | `#f3f4f6` | Inline code background (gray-100) |

#### Dark Mode (`.dark` class on `<html>`)

| Variable | Value | Usage |
|----------|-------|-------|
| `--color-bg` | `#111827` | Page background (gray-900) |
| `--color-text` | `#f9fafb` | Primary text (gray-50) |
| `--color-text-muted` | `#9ca3af` | Secondary text (gray-400) |
| `--color-accent` | `#60a5fa` | Links, buttons (blue-400) |
| `--color-accent-hover` | `#93c5fd` | Link hover state (blue-300) |
| `--color-border` | `#374151` | Borders, dividers (gray-700) |
| `--color-code-bg` | `#1f2937` | Inline code background (gray-800) |

### Typography

- **Font Stack**: System fonts (`system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`)
- **Content Width**: `max-w-2xl` (42rem / 672px)
- **Prose Styling**: Custom `.prose` class in `BlogPostLayout.astro`
- **Code Font**: `ui-monospace, SFMono-Regular, Menlo, monospace`

### Spacing Conventions

| Element | Spacing |
|---------|---------|
| Page padding | `px-6` (1.5rem horizontal) |
| Section gaps | `space-y-6` to `space-y-12` |
| Component margins | `my-4` to `my-8` |
| Max content width | `max-w-2xl mx-auto` |

### Dark Mode Implementation

The dark mode system works as follows:

1. **Detection** (in `BaseLayout.astro` inline script):
   - Check `localStorage.getItem('theme')`
   - Fall back to `prefers-color-scheme: dark`

2. **Application**:
   - Add/remove `dark` class on `<html>` element
   - CSS variables automatically update via `.dark` selector

3. **Toggle** (in `Header.astro`):
   - Button with sun/moon SVG icons
   - Saves preference to `localStorage`

4. **Code Blocks**:
   - Shiki generates dual-theme output
   - CSS variables switch between `--shiki-light-*` and `--shiki-dark-*`

**Critical**: The theme detection script is `is:inline` and runs in `<head>` before body renders to prevent flash of wrong theme.

---

## Content Management

### Blog Post Schema

Defined in `src/content.config.ts`:

```typescript
{
  title: string,              // Required, displayed as h1
  description: string,        // Required, max 160 chars (for SEO)
  pubDate: Date,              // Required, ISO format (YYYY-MM-DD)
  updatedDate?: Date,         // Optional, shows "Updated on" text
  draft?: boolean             // Optional, default false
}
```

### Creating a New Blog Post

1. **Copy the template**:
   ```bash
   cp src/content/blog/_template.md src/content/blog/YYYY-MM-DD-your-slug.md
   ```

2. **Edit frontmatter**:
   ```yaml
   ---
   title: "Your Post Title"
   description: "A compelling description under 160 characters for SEO."
   pubDate: 2024-01-15
   # updatedDate: 2024-01-20  # Uncomment if updating later
   # draft: true               # Uncomment to hide in production
   ---
   ```

3. **Write content** using supported markdown features

4. **Preview**: `npm run dev` → visit `http://localhost:4321/blog/your-slug`

5. **Publish**: `git add . && git commit -m "Add post" && git push`

### File Naming Convention

**Format**: `YYYY-MM-DD-slug.md`

| Part | Description | Example |
|------|-------------|---------|
| `YYYY` | 4-digit year | `2024` |
| `MM` | 2-digit month | `01` |
| `DD` | 2-digit day | `15` |
| `slug` | URL-friendly title | `my-first-post` |

**Examples**:
- `2024-01-15-hello-world.md` → `/blog/2024-01-15-hello-world`
- `2024-12-25-christmas-special.md` → `/blog/2024-12-25-christmas-special`

### Supported Markdown Features

#### Basic Formatting

```markdown
**bold** and *italic* and ~~strikethrough~~

- Bullet lists
- With multiple items

1. Numbered lists
2. Work too

> Blockquotes for emphasis

[Links](https://example.com)

![Images](/path/to/image.png)
```

#### Code Blocks

````markdown
Inline: `const x = 42;`

Block with syntax highlighting:
```typescript
interface User {
  name: string;
  email: string;
}
```
````

Supported languages: All major languages via Shiki (TypeScript, Python, Rust, Go, etc.)

#### LaTeX Mathematics

```markdown
Inline math: $E = mc^2$

Block math:
$$
\frac{d}{dx} \int_a^x f(t) dt = f(x)
$$
```

**Note**: LaTeX requires whitespace around `$` delimiters.

### Draft System

- Set `draft: true` in frontmatter to hide a post
- Drafts are **visible** in development (`npm run dev`)
- Drafts are **hidden** in production builds
- The filter logic is in `src/pages/blog/index.astro` and `src/pages/blog/[...slug].astro`

---

## Components Reference

### BaseHead.astro

**Location**: `src/components/BaseHead.astro`

**Props**:
| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `title` | `string` | Required | Page title for `<title>` tag |
| `description` | `string` | Required | Meta description for SEO |
| `needsMath` | `boolean` | `false` | Load KaTeX CSS if true |

**Usage**:
```astro
<BaseHead title="My Page" description="Page description" needsMath={true} />
```

### Header.astro

**Location**: `src/components/Header.astro`

**Props**: None

**Contains**:
- Site title link (→ `/`)
- Navigation: About, Blog
- Dark mode toggle button

**Dark Mode Toggle**:
- Uses inline SVGs (Heroicons sun/moon)
- JavaScript toggles `dark` class on `document.documentElement`
- Persists to `localStorage`

### Footer.astro

**Location**: `src/components/Footer.astro`

**Props**: None

**Contains**:
- Copyright notice with current year
- `SocialLinks` component

### SocialLinks.astro

**Location**: `src/components/SocialLinks.astro`

**Props**:
| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `showGitHub` | `boolean` | `true` | Show GitHub icon |
| `showLinkedIn` | `boolean` | `true` | Show LinkedIn icon |

**Configured URLs** (hardcoded in component):
- GitHub: `https://github.com/niarenaw`
- LinkedIn: `https://linkedin.com/in/nicholas-arena`

### BaseLayout.astro

**Location**: `src/layouts/BaseLayout.astro`

**Props**:
| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `title` | `string` | Required | Page title |
| `description` | `string` | Required | Meta description |
| `needsMath` | `boolean` | `false` | Pass to BaseHead |

**Structure**:
```html
<!doctype html>
<html>
  <head>
    <BaseHead ... />
  </head>
  <body>
    <Header />
    <main>
      <slot />  <!-- Page content goes here -->
    </main>
    <Footer />
    <script>  <!-- Theme detection -->
  </body>
</html>
```

### BlogPostLayout.astro

**Location**: `src/layouts/BlogPostLayout.astro`

**Props**:
| Prop | Type | Description |
|------|------|-------------|
| `title` | `string` | Post title (h1) |
| `description` | `string` | For SEO |
| `pubDate` | `Date` | Publication date |
| `updatedDate` | `Date?` | Optional update date |

**Features**:
- Extends `BaseLayout` with `needsMath={true}`
- Renders title as `<h1>`
- Shows publication/update dates
- Wraps content in `.prose` styling

---

## Configuration Files

### astro.config.mjs

```javascript
export default defineConfig({
  site: 'https://niarenaw.github.io',  // Used for canonical URLs
  vite: {
    plugins: [tailwindcss()],  // Tailwind 4 Vite plugin
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

**When to modify**:
- Adding new markdown plugins
- Changing code highlighting themes
- Updating the site URL

### src/content.config.ts

```typescript
const blog = defineCollection({
  loader: glob({ pattern: '**/[^_]*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    description: z.string().max(160),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    draft: z.boolean().default(false),
  }),
});
```

**When to modify**:
- Adding new frontmatter fields to blog posts
- Creating new content collections (e.g., projects, notes)
- Changing validation rules

### src/styles/global.css

**Structure**:
1. `@import "tailwindcss"` - Load Tailwind
2. `@custom-variant dark` - Define dark mode selector
3. `@theme` - CSS custom properties for light mode
4. `.dark` - Override properties for dark mode
5. Shiki theme rules - Code block styling
6. Utility classes - Inline code, etc.

**When to modify**:
- Changing brand colors
- Adding new CSS custom properties
- Adjusting dark mode colors
- Custom component styles

### tsconfig.json

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

**Path Alias**: Use `@/` instead of relative paths:
```typescript
import Header from '@/components/Header.astro';  // ✓
import Header from '../../../components/Header.astro';  // ✗
```

---

## Best Practices

### Code Style

1. **Use Tailwind utilities** over custom CSS when possible
2. **Use CSS variables** (`var(--color-*)`) for theming, not hardcoded colors
3. **Keep components small** - split if over 100 lines
4. **Use TypeScript** for all `.astro` frontmatter scripts
5. **Use path aliases** (`@/`) for imports

### Performance

1. **Conditional loading**: Only add `needsMath={true}` on pages with LaTeX
2. **Image optimization**: Use `public/` for static images, consider Astro's `<Image>` for optimization
3. **No unnecessary JS**: Avoid adding client-side JavaScript unless essential
4. **Minimize layout shift**: Set explicit dimensions on images

### Content

1. **Frontmatter validation**: Always include all required fields
2. **Description length**: Keep under 160 characters for SEO
3. **Date accuracy**: Use actual publication date, update `updatedDate` for major revisions
4. **Draft workflow**: Use `draft: true` while writing, remove when ready

### Accessibility

1. **Alt text**: Always provide for images
2. **Heading hierarchy**: Don't skip levels (h1 → h2 → h3)
3. **Link text**: Be descriptive (not "click here")
4. **Color contrast**: Use the defined CSS variables which meet WCAG standards

### Git Workflow

1. **Atomic commits**: One logical change per commit
2. **Test locally**: Run `npm run build` before pushing
3. **Preview production**: Use `npm run preview` to test the built site

---

## Common Tasks

### Adding a New Page

1. Create `src/pages/your-page.astro`
2. Import and use `BaseLayout`:
   ```astro
   ---
   import BaseLayout from '@/layouts/BaseLayout.astro';
   ---
   <BaseLayout title="Page Title" description="Description">
     <h1>Your Content</h1>
   </BaseLayout>
   ```
3. Add navigation link in `Header.astro` if needed

### Adding a New Component

1. Create `src/components/YourComponent.astro`
2. Define props interface:
   ```astro
   ---
   interface Props {
     title: string;
     optional?: boolean;
   }
   const { title, optional = false } = Astro.props;
   ---
   ```
3. Import where needed: `import YourComponent from '@/components/YourComponent.astro';`

### Changing Colors

1. Edit `src/styles/global.css`
2. Modify values in `@theme` block (light mode)
3. Update corresponding values in `.dark` block
4. Ensure sufficient contrast (use WebAIM contrast checker)

### Adding a New Blog Feature

Example: Adding tags to posts

1. **Update schema** in `src/content.config.ts`:
   ```typescript
   tags: z.array(z.string()).optional(),
   ```

2. **Update layout** in `BlogPostLayout.astro`:
   ```astro
   {tags && (
     <div class="flex gap-2">
       {tags.map(tag => <span class="...">{tag}</span>)}
     </div>
   )}
   ```

3. **Update existing posts** with new frontmatter field

### Running Development Server

```bash
npm run dev      # Start on http://localhost:4321
```

### Building for Production

```bash
npm run build    # Output to /dist/
npm run preview  # Preview production build locally
```

---

## Troubleshooting

### Build Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Invalid frontmatter" | Missing required field | Check Zod schema in `content.config.ts` |
| "Cannot find module" | Wrong import path | Use `@/` path alias |
| "Type error in .astro" | TypeScript issue | Check props interface |

### Styling Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Dark mode not working | Missing `.dark` class rules | Check `global.css` |
| Colors wrong | Using hardcoded values | Use `var(--color-*)` |
| Layout broken on mobile | Missing responsive classes | Add `sm:`, `md:` prefixes |

### Content Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Post not showing | `draft: true` in prod | Set `draft: false` |
| LaTeX not rendering | Missing `needsMath` | Add `needsMath={true}` to layout |
| Code not highlighted | Wrong language tag | Check Shiki supported languages |

### Development Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| HMR not working | Astro cache | Delete `.astro/` folder |
| Old content showing | Browser cache | Hard refresh (Cmd+Shift+R) |
| Port in use | Another process | Kill process or use `--port 4322` |

---

## Quick Reference

### NPM Scripts

```bash
npm run dev      # Development server
npm run build    # Production build
npm run preview  # Preview production
```

### File Locations

| Task | File |
|------|------|
| Add navigation | `src/components/Header.astro` |
| Change colors | `src/styles/global.css` |
| Edit about page | `src/pages/index.astro` |
| Add blog post | `src/content/blog/YYYY-MM-DD-slug.md` |
| Change SEO defaults | `src/components/BaseHead.astro` |
| Configure build | `astro.config.mjs` |

### CSS Variable Quick Reference

```css
var(--color-bg)           /* Background */
var(--color-text)         /* Primary text */
var(--color-text-muted)   /* Secondary text */
var(--color-accent)       /* Links, buttons */
var(--color-accent-hover) /* Hover states */
var(--color-border)       /* Borders */
var(--color-code-bg)      /* Code backgrounds */
```

---

*Last updated: 2024*
