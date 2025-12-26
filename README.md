# Personal Website

Personal website and technical blog built with [Astro](https://astro.build) and [Tailwind CSS](https://tailwindcss.com).

## Development

```bash
npm install
npm run dev
```

Site runs at `http://localhost:4321`.

## Adding a Blog Post

1. Create a new file in `src/content/blog/` with format `YYYY-MM-DD-slug.md`
2. Add frontmatter:
   ```yaml
   ---
   title: "Post Title"
   description: "Brief description (max 160 chars)"
   pubDate: 2025-01-15
   ---
   ```
3. Write content using markdown, code blocks, and LaTeX (`$inline$` or `$$block$$`)
4. Preview with `npm run dev`

## Deployment

Pushes to `main` automatically deploy to GitHub Pages via GitHub Actions.

### First-time Setup

1. Go to repository Settings > Pages
2. Set Source to "GitHub Actions"
3. Push to `main`

## Notifying Subscribers

When publishing a **new** blog post (not edits to existing posts):

1. Push the new post to `main` branch
2. Wait for deployment to complete
3. Go to [Buttondown dashboard](https://buttondown.email/emails)
4. Click "New email"
5. Write notification:
   ```
   Subject: New post: [Post Title]

   Body:
   I just published a new post: [Post Title]

   [Brief 1-2 sentence description]

   Read it here: https://nicharena.com/blog/[slug]
   ```
6. Preview and send

**Important:** Only notify for new posts. Edits to existing posts (typos, clarifications) do not warrant notifications.
