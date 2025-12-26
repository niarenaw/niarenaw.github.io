# RSS Automation - Stashed for Future Use

Stashed on: 2025-12-25

## Why Stashed
Buttondown RSS-to-email requires paid tier. Keeping this for future implementation.

---

## File: `/src/pages/rss.xml.js`

```javascript
import rss from '@astrojs/rss';
import { getCollection } from 'astro:content';

export async function GET(context) {
  const posts = await getCollection('blog', ({ data }) => !data.draft);

  return rss({
    title: 'Nich Arena',
    description: 'Blog posts about my learnings - mostly math.',
    site: context.site,
    items: posts
      .sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf())
      .map((post) => ({
        title: post.data.title,
        pubDate: post.data.pubDate,
        description: post.data.description,
        link: `/blog/${post.id}/`,
      })),
  });
}
```

## Package Required

```bash
npm install @astrojs/rss
```

## Buttondown RSS-to-Email Setup (when ready)

1. Go to Buttondown Settings → RSS-to-email
2. Enter: `https://nicharena.com/rss.xml`
3. Set frequency: "Immediately"
4. Enable "Skip old items"
5. Save

## Alternative: GitHub Actions + Buttondown API

If Buttondown RSS-to-email stays paid, can use their free API instead:
- Trigger on deploy
- Detect new posts via git diff
- POST to `https://api.buttondown.email/v1/emails`
