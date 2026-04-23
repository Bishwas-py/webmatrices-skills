---
name: trending-topics
description: Find trending content ideas from Reddit, fact-check with live data, and draft a Webmatrices post. Use when asked about trending topics, content ideas, what to write about, or what's hot.
disable-model-invocation: true
argument-hint: [topic-or-segment]
---

# Trending Topics

Find trending content ideas from Reddit, verify with live data, and draft a publish-ready Webmatrices post.

For shared research workflows and viral content patterns, see [content-engine.md](../_shared/content-engine.md).

## Arguments

- **No args** → broad scan (browse hot posts from all Tier 1 subs)
- **Segment keyword** (`adsense`, `fiverr`, `seo`, `webdev`, `saas`, `ai`) → targeted subreddits per mapping in content-engine.md
- **Free text** (e.g. "adsense rejection rates") → Reddit search query across all subs

## Workflow

### Step 1: Parse the argument

Check `$ARGUMENTS`:
- Empty → broad scan mode
- Matches a keyword (adsense/fiverr/seo/webdev/saas/ai) → targeted mode with mapped subs from content-engine.md
- Anything else → free text search mode

### Step 2: Reddit Scout

**Broad scan mode:**
For each Tier 1 subreddit (Adsense, Blogging, SEO, webdev, SaaS):
```
browse_subreddit(subreddit, sort="hot", limit=10)
```

**Targeted mode:**
```
search_reddit(query=keyword, subreddits=mapped_subs, sort="hot", time="week", limit=15)
```

**Free text mode:**
```
search_reddit(query=free_text, sort="relevance", time="month", limit=20)
```

Filter results: keep only posts with score >= 5 AND comments >= 3.

### Step 3: Rank and present

Apply the ranking formula from content-engine.md:
```
engagement_score = (upvotes * 0.4) + (num_comments * 0.6)
posts < 48h old get 1.5x multiplier
```

Present top 5-8 results to user as a table:

| # | Title | Subreddit | Score | Comments | Angle |
|---|-------|-----------|-------|----------|-------|

For each, include a one-line suggested angle for a Webmatrices post.

Ask user to pick one (or request more results).

### Step 4: Deep dive

On the user's pick, run `get_post_details` for the chosen post.

Extract:
- Key claims with specific numbers
- Top comment insights (often better than OP)
- URLs mentioned in thread
- Pain points and unanswered questions
- The contrarian angle or surprising insight

Present a summary: "here's what this thread is really about, and here's the angle I'd take."

### Step 5: Fact-check and research

Follow the fact-checking workflow from content-engine.md:

1. **WebSearch** — 2-3 queries to verify the key claims from the thread
   - Is this policy change real? When did it happen?
   - Are the numbers accurate? What do other sources say?
   - Any newer data that updates or contradicts the thread?

2. **Playwright** (only if the thread contains URLs worth crawling)
   - Crawl linked pages for stats, evidence, screenshots
   - Extract specific data points to strengthen the post

3. Flag anything unverifiable — frame as "reddit users report" not fact

Present findings: "here's what checked out, here's what didn't, here's new data I found."

### Step 6: Draft the post

1. **Pick persona** — use audience-matcher logic:
   - Fetch all personas from `get_self_personas` MCP
   - Match topics to personas by comparing against each persona's `metadata.personaTraits.topics` array
   - Score by topic relevance, voice fit, and backstory relevance using audience-matcher logic

2. **Apply viral patterns** from content-engine.md:
   - Jagged number in title
   - Hook in first line (fact or tension)
   - Personal angle + data/evidence
   - Contrarian insight
   - Closing question

3. **Format:**
   - HTML body (`<p>`, `<h2>`, `<strong>`, `<ol>`, `<ul>`)
   - 300-500 words
   - Include an excerpt (1-2 sentences)
   - Lowercase title

4. **Credit sources** — reference Reddit discussion naturally ("someone on r/Adsense pointed out..."), don't link directly to Reddit threads.

Present the full draft to user:
- Persona chosen (and why)
- Title
- Excerpt
- Full HTML body
- Ask for approval or edits

### Step 7: Hand off to publish-post

On approval, tell user:
> "Draft ready. Run `/webmatrices:publish-post [persona-username]` to publish, or edit the draft first."

Do NOT insert into the database directly. The publish-post skill handles DB insertion.
