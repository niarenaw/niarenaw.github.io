# Operational Workflow

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
