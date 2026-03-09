---
name: publish-post
description: Write and publish a post on Webmatrices as a specific user persona. Use when asked to create a post, write content, publish an article, or seed content.
disable-model-invocation: true
argument-hint: [username] [topic]
---

# Publish Post

Write and publish a post on Webmatrices as a specific user persona.

## Database Connection

All scripts MUST run from `~/Projects/webm-frontend` for Prisma access. Read production credentials from `.env`:

```bash
cd ~/Projects/webm-frontend
DATABASE_URL=$(grep '^PROD_DATABASE_URL=' .env | cut -d'"' -f2) node temp-script.mjs
```

## Writing Style Guidelines

Match the voice of the persona being used. General principles across all Webmatrices personas:

- **Lowercase titles** — no title case, feels more authentic
- **First-person** — "i did this", "here's what happened"
- **Real numbers** — specific figures, not vague claims
- **Conversational** — write like you're talking to a friend on a forum
- **End with a question** — drives engagement and replies
- **HTML body** — use `<p>`, `<h2>`, `<strong>`, `<ol>`, `<ul>`, `<code>` tags
- **No emojis** in body text unless it fits the persona
- **Include an excerpt** — 1-2 sentence summary for previews

## Post Schema

```typescript
{
  title: string,           // lowercase, punchy
  slug: string,            // kebab-case from title
  body: string,            // HTML content
  excerpt: string,         // 1-2 sentences
  authorId: number,        // user ID of the persona
  publishedAt: new Date(), // set to now
}
```

## Workflow

1. Identify which persona to use:
   - If username provided as `$0`, look up that user
   - Otherwise, suggest the best-matching persona for the topic
2. If topic provided as `$1`, use it. Otherwise, ask what to write about
3. Research the topic if needed (use WebSearch for current data/stats)
4. Write the post in the persona's voice
5. Show the draft to the user for approval
6. On approval, insert into the database
7. Display the post ID, slug, and title
8. Clean up temporary scripts

## Important

- Always verify the persona user exists and is active before publishing
- Generate a unique slug from the title
- Set `publishedAt` to now so it appears immediately
- The body should be substantive (300+ words for opinion/news pieces)
