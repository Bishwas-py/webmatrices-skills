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

## Database & API Credentials

All scripts MUST run from `~/Projects/webm-frontend` for Prisma access. Credentials are read from `~/Projects/webm-frontend/.env`:

- `PROD_DATABASE_URL` — production PostgreSQL connection string
- `BREVO_API_KEY` — Brevo email API key

Extract and pass them when running scripts:
```bash
cd ~/Projects/webm-frontend
DATABASE_URL=$(grep '^PROD_DATABASE_URL=' .env | cut -d'"' -f2) \
BREVO_API_KEY=$(grep '^BREVO_API_KEY=' .env | cut -d'"' -f2) \
node temp-script.mjs
```

## Workflow

### Step 1: Fetch the post and its context

Query the post by slug or ID. Include:
- Post title, body, author info
- All existing comments with their authors, likes, and nested replies
- Identify which comments are unanswered

```typescript
const post = await prisma.post.findFirst({
  where: { OR: [{ slug: identifier }, { id: identifier }], deletedAt: null },
  include: {
    author: { select: { username: true } },
    comments: {
      where: { deletedAt: null },
      include: {
        user: { select: { username: true, email: true } },
        replies: { include: { user: { select: { username: true } } } }
      },
      orderBy: { createdAt: 'asc' }
    }
  }
});
```

### Step 2: Understand what the user is asking

Categorize the post:

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

1. Use `search_reddit` with the post's core topic (e.g. "adsense low value content rejection")
2. Target 2-3 relevant subreddits from the mapping in content-engine.md
3. Pull `get_post_details` on 1-2 high-engagement threads
4. Extract:
   - Real user experiences and data points
   - Common advice being given (to either agree with or counter)
   - Any recent changes or news referenced

Use these Reddit insights to:
- Add real-world context to the reply ("saw someone on r/Adsense deal with the same thing...")
- Back up advice with community evidence
- Identify contrarian angles the user hasn't considered

### Step 3: Research (choose the right tool)

#### When to use Playwright (URL present)

Use Playwright ONLY when the user shared a URL that needs live analysis:

**AdSense site review:**
```
1. Navigate to the user's site URL
2. Check: homepage content, navigation, about page, privacy policy, contact page
3. Count articles/pages, assess content depth
4. Check mobile responsiveness
5. Look for: thin content, duplicate titles, missing legal pages, ad placement issues
6. Take a screenshot for reference
```

**Fiverr gig review:**
```
1. Navigate to the Fiverr gig URL
2. Extract: title, description, pricing tiers, delivery time, tags
3. Check: thumbnail quality, portfolio samples, seller stats (level, rating, reviews)
4. Compare against top competitors in same search
5. Take a screenshot for reference
```

#### When to use WebSearch only (no URL or general question)

Run 1-3 targeted searches:
- Current Google policies/algorithm changes
- Fiverr algorithm updates or seller tips
- SEO best practices for the specific sub-topic
- Technical documentation for coding questions

### Step 4: Pick the persona

If persona provided as `$1`, use that user. Otherwise, use the audience-matcher logic:

| Topic | Best Persona | Why |
|-------|-------------|-----|
| AdSense revenue/changes | warmreboot (27378) | anxious dev who deals with this |
| AdSense site review | bishwasbhn (1) | admin authority, "I crawled your site" |
| Fiverr gig review | simonokimo (26714) or techwizardrino (19150) | simonokimo does deep gig audits, techwizardrino gives buyer perspective |
| SEO strategy | serpsherpa (19191) or digitaldave01 (8653) | serpsherpa for storytelling, digitaldave01 for practical |
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
- Lowercase everything (titles, sentences). No Title Case.
- First person: "i looked at your site", "i've seen this before"
- Real numbers: "your 4.8 rating on that gig is dragging your average"
- Slight frustration or urgency: "nobody tells you this but..."
- Short paragraphs. No walls of text.
- HTML format: `<p>`, `<strong>`, `<ol>`, `<ul>`, `<code>`, `<a>` tags
- Max 300-500 words for reviews, 100-200 for discussion replies

**Apply viral content patterns from content-engine.md:**
- Lead with what's specific to THEIR situation
- Include a data point from Reddit research or fact-checking
- One contrarian insight they didn't expect
- Close with a question that invites follow-up

**What to NEVER do:**
- Generic advice lists ("1. Create quality content 2. Improve SEO 3. Be patient")
- ChatGPT voice ("I'd be happy to help!", "Great question!", "Here's a comprehensive guide")
- Empty acknowledgments ("thanks for sharing", "interesting post")
- Placeholder promises ("looking into it", "will get back to you")

### Step 6: Present draft and confirm

Show the user:
- The chosen persona
- The target (post or specific comment being replied to)
- The full draft in rendered form
- Ask for approval or edits

### Step 7: Insert the reply AND send email notification

On approval, create the comment, update post stats, AND send the notification email — all in a single script. The email notification is **mandatory** — every reply must notify the post author (or parent comment author for nested replies).

```javascript
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient({ datasources: { db: { url: process.env.DATABASE_URL } } });

const postId = '<post-id>';
const personaId = <persona-user-id>;

const htmlContent = `<p>your reply here...</p>`;

// 1. Insert comment
const comment = await prisma.comment.create({
  data: {
    content: htmlContent,
    userId: personaId,
    postId: postId,
    // parentId: commentId  ← only if replying to a specific comment
  }
});

// 2. Update post comment count and activity
await prisma.post.update({
  where: { id: postId },
  data: {
    commentCount: { increment: 1 },
    lastActivityAt: new Date()
  }
});

// 3. Fetch post author info for email
const post = await prisma.post.findUnique({
  where: { id: postId },
  include: { author: { select: { id: true, username: true, email: true } } }
});

// 4. Get persona username for email
const persona = await prisma.user.findUnique({
  where: { id: personaId },
  select: { username: true }
});

// 5. Send notification email via Brevo API
const siteUrl = 'https://webmatrices.com';
const targetUrl = `${siteUrl}/comment/${comment.id}`;
const settingsUrl = `${siteUrl}/settings/notifications`;
const commenterUsername = persona.username;
const postTitle = post.title;

// Strip HTML and truncate for email preview
const commentExcerpt = htmlContent.replace(/<[^>]*>/g, '').slice(0, 200) + '...';

const subject = `New comment on your post "${postTitle.slice(0, 40)}"`;

const emailHtml = `<!DOCTYPE html><html><body style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; background: #f5f5f5; padding: 20px;">
<div style="max-width: 600px; margin: 0 auto; background: white; border-radius: 8px; padding: 32px;">
  <h1 style="margin: 0 0 16px 0; font-size: 24px; font-weight: 700; color: #1a1a1a;">${commenterUsername} commented on your post</h1>
  <p style="margin: 0 0 24px 0; font-size: 16px; color: #666;">Hi ${post.author.username},</p>
  <p style="margin: 0 0 24px 0; font-size: 16px; color: #666;"><strong style="color: #1a1a1a;">${commenterUsername}</strong> left a comment on your post "<strong style="color: #1a1a1a;">${postTitle}</strong>".</p>
  <div style="background: #f9f9f9; border-left: 4px solid #22c55e; border-radius: 0 8px 8px 0; padding: 16px 20px; margin: 0 0 24px 0;">
    <p style="margin: 0 0 8px 0; font-size: 13px; font-weight: 600; color: #999; text-transform: uppercase;">Comment</p>
    <p style="margin: 0; font-size: 15px; color: #1a1a1a; line-height: 1.6; font-style: italic;">"${commentExcerpt}"</p>
  </div>
  <a href="${targetUrl}" style="display: inline-block; padding: 12px 24px; background: #22c55e; color: white; text-decoration: none; border-radius: 6px; font-weight: 600;">View Comment</a>
  <hr style="margin: 32px 0 16px 0; border: none; border-top: 1px solid #eee;">
  <p style="font-size: 12px; color: #999;">You received this because you have email notifications enabled. <a href="${settingsUrl}" style="color: #999;">Manage preferences</a></p>
</div>
</body></html>`;

const emailText = `${commenterUsername} commented on your post\n\nHi ${post.author.username},\n\n${commenterUsername} left a comment on your post "${postTitle}":\n\n"${commentExcerpt}"\n\nView Comment: ${targetUrl}\n\nManage notifications: ${settingsUrl}`;

const response = await fetch('https://api.brevo.com/v3/smtp/email', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
    'api-key': process.env.BREVO_API_KEY
  },
  body: JSON.stringify({
    sender: { name: 'Webmatrices', email: 'noreply@webmatrices.com' },
    to: [{ email: post.author.email, name: post.author.username }],
    subject,
    htmlContent: emailHtml,
    textContent: emailText
  })
});

if (!response.ok) {
  console.error('Brevo error:', await response.text());
} else {
  const result = await response.json();
  console.log(`Email sent to ${post.author.email}:`, result);
}

await prisma.$disconnect();
```

**For replies to comments** (not the post), change the email to notify the parent comment author instead:
- Use `type: 'comment_reply'` in the subject
- Fetch `comment.parent.user` for the recipient
- Subject: `New reply to your comment on "${postTitle}"`

### Step 8: Clean up

Remove any temporary scripts created during execution.

## Replying to a specific comment

If the user wants to reply to a specific comment (not the post itself):
- Show all comments on the post with IDs
- Let the user pick which comment to reply to
- Set `parentId` to that comment's ID
- Reference the parent comment's author by username in the reply

## Batch mode

If invoked with `--batch` or asked to "reply to all unanswered posts":
1. Query posts that have real user comments but no admin/persona replies
2. For each, generate a draft
3. Present all drafts for approval
4. Insert approved replies one by one

## Additional resources

- For persona details, see the [audience-matcher skill](../audience-matcher/SKILL.md)
- For existing reply patterns and what gets liked, see [reply-patterns.md](reply-patterns.md)
