---
name: publish-post
description: Write and publish a post on Webmatrices as a specific user persona. Use when asked to create a post, write content, publish an article, or seed content.
disable-model-invocation: true
argument-hint: [username] [topic]
---

# Publish Post

Write and publish a post on Webmatrices as a specific user persona using MCP tools.

## Writing Style Guidelines

Match the voice of the persona being used. General principles across all Webmatrices personas:

- **Lowercase titles** — no title case, feels more authentic
- **First-person** — "i did this", "here's what happened"
- **Real numbers** — specific figures, not vague claims
- **Conversational** — write like you're talking to a friend on a forum
- **End with a question** — drives engagement and replies
- **HTML body** — use `<p>`, `<h2>`, `<strong>`, `<ol>`, `<ul>`, `<code>` tags
- **No emojis** in body text unless it fits the persona
- **300+ words** for opinion/news pieces

## Workflow

1. Identify which persona to use:
   - If username provided, use `get_user_details` with `username` to look them up
   - Otherwise, suggest the best-matching persona for the topic
2. If topic provided, use it. Otherwise, ask what to write about
3. Research the topic if needed (use WebSearch for current data/stats)
4. Write the post in the persona's voice
5. Show the draft to the user for approval
6. On approval, use `create_post` with:
   - `authorId`: the persona's user ID
   - `title`: lowercase, punchy
   - `body`: full HTML content
   - `tagSlugs`: relevant tag slugs (e.g., `["google-adsense", "digi-work"]`)
   - `createdAt`: ISO date string (optional, for backdating)
7. Display the result (post ID, slug)

## Staggering Multiple Posts

When publishing multiple posts in one session, **never** publish them all at the same timestamp. Stagger them to look organic:

- **2 posts:** 4-8 hours apart
- **3 posts:** spread across 2-3 different days
- **4+ posts:** spread across 3-5 days, max 2 per day

Use the `createdAt` parameter on `create_post` to set different publish dates. Pick realistic times (not 3am unless the persona is a night owl). Good defaults:
- Morning posts: 08:00-10:00 UTC
- Afternoon posts: 14:00-16:00 UTC
- Evening posts: 19:00-21:00 UTC

Example for 3 posts published today (March 29):
```
Post 1: createdAt = "2026-03-27T09:30:00Z"
Post 2: createdAt = "2026-03-28T15:00:00Z"
Post 3: createdAt = "2026-03-29T11:00:00Z"
```

Different personas should post at different times. Dont have techwizardrino and romanking both post at 9am.

## Available Tags

Use `list_tags` to get current tags. Common ones:
- `google-adsense` — AdSense topics
- `digi-work` — Freelancing/Fiverr/Upwork
- `programming` — Coding/tech
- `ai-founder` — AI tools/startups
- `sveltekit` — SvelteKit framework
- `django` — Django framework

## Important

- The MCP `create_post` tool runs content moderation (`guardContent`) automatically
- It generates unique slugs, updates tag counts, and creates audit logs
- Always verify the persona exists and is active before publishing
- Show the draft for approval before creating
