---
name: find
description: Search posts, users, tags, and analytics on the Webmatrices database. Use when asked to find posts, search users, check engagement, look up tags, or run any read-only database query.
disable-model-invocation: true
argument-hint: [natural-language-query]
---

# Find

Search and query the Webmatrices production database using natural language. Replaces the old `db-query` and `fetch-users` skills with a single, smarter interface.

## Database Connection

All scripts MUST run from `~/Projects/webm-frontend` for Prisma access. Read production credentials from `.env`:

```bash
cd ~/Projects/webm-frontend
DATABASE_URL=$(grep '^PROD_DATABASE_URL=' .env | cut -d'"' -f2) node temp-script.mjs
```

## Schema Reference

### Core models

- **User** (`users`): id (Int), username, email, password, firstName, lastName, isActive, isStaff, isSuperuser, isPremium, dateJoined, lastLogin
- **Profile** (`profiles`): userId, bio, image, city, region, country, countryCode, continent, timezone
- **Post** (`posts`): id (String/cuid), title, slug, body, excerpt, featuredImage, authorId (Int), viewCount*, likeCount, commentCount, createdAt, updatedAt, lastActivityAt, publishedAt, deletedAt
- **Comment** (`comments`): id (String/cuid), content (HTML), userId (Int), postId, tagId, parentId (self-ref for replies), likeCount, createdAt, deletedAt
- **Tag** (`tags`): id (String/cuid), name, slug, description, iconifyString, userId, postCount, viewCount, createdAt
- **Impression** (`impressions`): id, postId, commentId, tagId, userId, viewerIp, countryCode, country, viewCount, totalDuration, startedAt, lastActiveAt
- **PostLike** (`post_likes`): id, userId, postId, createdAt
- **CommentLike** (`comment_likes`): id, userId, commentId, createdAt

### Other models (queryable but less common)

- **Playbook**, **PlaybookChapter**, **PlaybookPurchase**, **PlaybookDiscussion**
- **Checklist**, **CustomChecklist**, **Project**, **ProjectMember**
- **EmailCampaign**, **EmailRecipient**, **EmailClick**
- **ToolPurchase**

### Critical data note

**`Post.viewCount` is always 0.** Real view data lives in the `impressions` table. When views or engagement ratios are needed, you MUST aggregate from impressions:

```typescript
const viewData = await prisma.impression.groupBy({
  by: ['postId'],
  where: { postId: { not: null } },
  _sum: { viewCount: true },
  orderBy: { _sum: { viewCount: 'desc' } }
});
```

## Safety Rules

- **READ ONLY** — never run UPDATE, DELETE, or INSERT through this skill
- Always filter out soft-deleted records (`deletedAt: null`) unless explicitly asked for deleted records
- Never expose full password hashes in output
- Never expose user API keys (openAiApiKey, scraperApiKey) in output

## Query Detection

Parse the natural language input and detect the **entity type** and **filter type**:

### Post queries

| Input pattern | Detection | Prisma approach |
|---------------|-----------|-----------------|
| `"fiverr posts"`, `"posts about adsense"` | keyword search | `title/body contains` (case-insensitive) |
| `"unanswered posts"`, `"posts with no replies"` | status filter | `commentCount: 0` |
| `"posts needing staff reply"` | status + join | Posts with comments but no comment from `isStaff: true` user |
| `"high engagement posts"` | metric sort | Join impressions for real views, compute (likes+comments)/views |
| `"trending posts"`, `"popular this week"` | metric + time | Impressions from last 7 days, sorted by view sum |
| `"posts from last 7 days"` | time filter | `createdAt >= 7d ago` |
| `"posts by @username"` | author filter | `author.username = value` |
| `"posts tagged adsense"` | tag filter | `tags: { some: { name: { contains: value, mode: 'insensitive' } } }` |
| `"fiverr posts needing replies"` | keyword + status | Combine keyword search with status filter |

### User queries

| Input pattern | Detection | Prisma approach |
|---------------|-----------|-----------------|
| `"users with email bishwasbh"` | email prefix | `email: { startsWith: value }` |
| `"user @username"`, `"find user john"` | username | `username: { contains: value, mode: 'insensitive' }` |
| `"user #12345"`, `"user id 12345"` | user ID | `id: value` |
| `"recent users"` | time sort | `orderBy: { dateJoined: 'desc' }, take: 20` |
| `"active users"`, `"inactive users"` | status | `isActive: true/false` |
| `"staff users"` | role | `isStaff: true` |
| `"banned users"`, `"spammers"` | status | `isActive: false`, ordered by most recent |
| `"premium users"` | premium | `isPremium: true` |
| `"users from Nepal"`, `"users in US"` | geo | Join profile, filter by country/countryCode |

### Tag queries

| Input pattern | Detection | Prisma approach |
|---------------|-----------|-----------------|
| `"top tags"`, `"popular tags"` | metric sort | `orderBy: { postCount: 'desc' }` |
| `"all tags"` | list all | `findMany` |
| `"tag adsense"` | specific tag | `name: { contains: value, mode: 'insensitive' }` |

### Analytics / aggregate queries

| Input pattern | Detection | Prisma approach |
|---------------|-----------|-----------------|
| `"how many posts"`, `"count users"` | count | `prisma.model.count(...)` |
| `"posts per tag"` | groupBy | Aggregate posts by tag relation |
| `"views by country"` | groupBy | `impression.groupBy({ by: ['countryCode'] })` |
| `"engagement over time"` | time series | Group by week/month |
| Any other analytical question | fallback | Translate directly to Prisma query |

### Combining filters

Queries can combine multiple filter types. Parse all applicable filters:

- `"fiverr posts with high engagement"` → keyword("fiverr") + metric sort(engagement)
- `"unanswered adsense posts from last month"` → keyword("adsense") + status(commentCount:0) + time(30d)
- `"active users from India"` → status(isActive) + geo(India)

## Output Format

### Posts table

```
| # | Title | Author | Tags | Likes | Comments | Views* | Age |
|---|-------|--------|------|-------|----------|--------|-----|
| 1 | AdSense revenue drop: $1.5K to... | @techwizardrino | Google Adsense | 4 | 12 | 3,578 | 45d |
```

*Views from impressions table. Show "—" if no impression data.

Include a body preview (first 100 chars, stripped of HTML) when showing ≤5 posts.

### Users table

```
| # | ID | Username | Email | Active | Staff | Posts | Comments | Joined |
|---|-----|----------|-------|--------|-------|-------|----------|--------|
| 1 | 27378 | warmreboot | warm@... | Yes | No | 1 | 0 | 2025-... |
```

### Tags table

```
| # | Name | Posts | Views |
|---|------|-------|-------|
| 1 | AI Founder | 565 | 0 |
```

### Aggregates

Display as simple key-value pairs or a compact table depending on the data shape.

### Always include

- Total count at the bottom: `**Found 23 posts matching "fiverr"**`
- Default limit: 20 results. Show more if user asks.
- Sort order used: mention what the results are sorted by

## Workflow

1. Parse the natural language query — detect entity type(s) and filter(s)
2. Build the Prisma query — use the detection tables above
3. If views/engagement needed, run a separate impressions aggregation and join in-memory
4. Write a temporary script in the project directory, execute with `DATABASE_URL` set
5. Format results as a markdown table
6. Display results with total count and sort order
7. Clean up temporary scripts

## Current Platform Stats (for context)

- 161 active posts, 42 unanswered, 32 with comments but no staff reply
- 27,312 users (27,241 active, 71 banned, 3 staff)
- 7 tags: AI Founder, Digi Work, Programming, General, Google Adsense, SvelteKit, Django
- 116,808 impressions
- Top topics: adsense (46 posts), fiverr (23 posts), seo (22 posts)

## Example Queries

```
/webmatrices:find "fiverr posts"
/webmatrices:find "unanswered posts about adsense"
/webmatrices:find "high engagement posts"
/webmatrices:find "trending posts this week"
/webmatrices:find "posts needing staff reply"
/webmatrices:find "users with email bishwasbh"
/webmatrices:find "banned users"
/webmatrices:find "recent users from India"
/webmatrices:find "top tags"
/webmatrices:find "how many posts per tag"
/webmatrices:find "views by country"
```
