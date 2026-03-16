---
name: smart-reply
description: Draft a high-quality reply to a Webmatrices post or comment. Researches the topic, optionally crawls user URLs with Playwright, and writes in the chosen persona's voice. Use when asked to reply to a post, respond to a comment, or answer a user's question.
disable-model-invocation: true
argument-hint: [post-slug-or-id] [persona-username]
---

# Smart Reply

Draft an authentic, high-quality reply to a Webmatrices post or comment that matches the engagement patterns people actually appreciate on the platform.

For shared research workflows and viral content patterns, see [content-engine.md](../_shared/content-engine.md).

## What Makes a Good Reply (data-driven)

Analysis of 359 comments shows replies that get 2-5 likes share these traits:

1. **"I actually looked at YOUR thing"** — reference the user's specific site/gig/numbers, not generic advice
2. **Lead with what's working** before criticism — "Level 2, 24 reviews, your fundamentals are solid. Here's what's killing you..."
3. **One contrarian insight** the user didn't expect
4. **Specific data** — exact numbers, percentages, comparisons
5. **End with a direct question** — drives follow-up replies
6. **Lowercase, conversational tone** — never sound like ChatGPT

What gets **zero engagement**: "thanks for sharing", generic advice lists, link-only comments, placeholder responses like "looking into it".

## Workflow

### Step 1: Fetch the post and its context

Use `list_posts` with `search` to find the post by slug or title. Review existing comments and identify which are unanswered.

### Step 2: Understand what the user is asking

| Category | Signals | Action Required |
|----------|---------|-----------------|
| **AdSense help** (URL shared) | "rejected", "approval", "low value", site URL present | Playwright: crawl their site |
| **AdSense help** (no URL) | "rejected", "approval", general question | WebSearch for current AdSense policies |
| **Fiverr help** (URL shared) | "impressions", "clicks", "orders", Fiverr gig URL | Playwright: scrape their Fiverr gig |
| **Fiverr help** (no URL) | "impressions", "clicks", general Fiverr question | WebSearch for Fiverr algorithm tips |
| **SEO question** | "ranking", "backlinks", "traffic", "core update" | WebSearch for current SEO data |
| **Tech/coding** | code snippets, framework names, error messages | WebSearch + codebase knowledge |
| **Discussion/opinion** | thought pieces, debates, hot takes | WebSearch for supporting data/counterpoints |

### Step 2b: Reddit context check

After categorizing the post, search Reddit for what people are currently saying about the same topic:

1. Use `search_reddit` with the post's core topic
2. Target 2-3 relevant subreddits from the mapping in content-engine.md
3. Pull `get_post_details` on 1-2 high-engagement threads
4. Extract real user experiences, common advice, and recent changes

### Step 3: Research (choose the right tool)

#### When to use Playwright (URL present)

Use Playwright ONLY when the user shared a URL that needs live analysis (site review, Fiverr gig scrape).

#### When to use WebSearch only (no URL or general question)

Run 1-3 targeted searches for current policies, algorithm changes, or best practices.

### Step 4: Pick the persona

If persona provided, use `get_user_details` to look them up. Otherwise:

| Topic | Best Persona | Why |
|-------|-------------|-----|
| AdSense revenue/changes | warmreboot (27378) | anxious dev who deals with this |
| AdSense site review | bishwasbhn (1) | admin authority, "I crawled your site" |
| Fiverr gig review | simonokimo (26714) or techwizardrino (19150) | deep gig audits / buyer perspective |
| SEO strategy | serpsherpa (19191) or digitaldave01 (8653) | storytelling / practical |
| Tech/coding | techwizardrino (19150) or bishwasbhn (1) | dev voices |
| Hot take/opinion | romanking (45) or digitaldave01 (8653) | sharp, contrarian |

### Step 5: Draft the reply

**Structure that works (based on top-liked comments):**

```
[Opening — acknowledge their specific situation, reference their data/site/gig]
[What's working — find something genuine to compliment]
[The real issue — 1-3 specific problems with evidence]
[Actionable steps — numbered, specific, not generic]
[Contrarian insight — something unexpected]
[Closing question — invites follow-up]
```

**Voice rules:**
- Lowercase everything. No Title Case.
- First person: "i looked at your site", "i've seen this before"
- Real numbers: "your 4.8 rating on that gig is dragging your average"
- Short paragraphs. No walls of text.
- HTML format: `<p>`, `<strong>`, `<ol>`, `<ul>`, `<code>`, `<a>` tags
- Max 300-500 words for reviews, 100-200 for discussion replies

**What to NEVER do:**
- Generic advice lists ("1. Create quality content 2. Improve SEO 3. Be patient")
- ChatGPT voice ("I'd be happy to help!", "Great question!")
- Empty acknowledgments ("thanks for sharing", "interesting post")

### Step 6: Present draft and confirm

Show the user the chosen persona, target post, and full draft. Ask for approval or edits.

### Step 7: Insert the reply using MCP

On approval, use `create_comment` with:
- `userId`: the persona's user ID
- `postId`: the target post ID
- `content`: the HTML reply
- `parentCommentId`: only if replying to a specific comment

The MCP `create_comment` tool automatically:
- Runs content moderation (`guardContent`)
- Updates post comment count and last activity
- Sends email notification to the post/comment author
- Creates audit log

### Step 8: Done

Display confirmation with the comment ID.

## Replying to a specific comment

If the user wants to reply to a specific comment (not the post itself):
- Show all comments on the post
- Let the user pick which comment to reply to
- Pass `parentCommentId` to `create_comment`

## Batch mode

If asked to "reply to all unanswered posts":
1. Use `list_posts` to find posts needing replies
2. For each, generate a draft
3. Present all drafts for approval
4. Use `create_comment` for each approved reply

## Additional resources

- For persona details, see the [audience-matcher skill](../audience-matcher/SKILL.md)
- For existing reply patterns and what gets liked, see [reply-patterns.md](reply-patterns.md)
