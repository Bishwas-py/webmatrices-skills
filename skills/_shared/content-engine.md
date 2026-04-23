# Content Engine — Shared Reference

Shared research workflows and content patterns used by multiple skills.
Referenced by: trending, write, simulate-engagement

## 1. Subreddit Mapping

Table mapping keywords to subreddits:
| Keyword | Subreddits |
|---------|-----------|
| adsense | Adsense, Blogging, juststart |
| fiverr | Fiverr, Freelancers, Entrepreneur |
| seo | SEO, Blogging, juststart |
| webdev | webdev, programming, vibecoding, ExperiencedDevs |
| saas | SaaS, Entrepreneur, EntrepreneurRideAlong |
| ai | ArtificialInteligence, vibecoding, programming |
| (none/broad) | Adsense, Blogging, SEO, webdev, SaaS |

## 2. Reddit Research Workflow

### When to browse vs search

- **No argument / broad scan:** Use `browse_subreddit` on each Tier 1 sub, sort by `hot`, limit 10 per sub
- **Keyword argument (adsense, seo, etc.):** Use `search_reddit` with keyword, mapped subs from table above, sort `hot`, time `week`
- **Free text argument:** Use `search_reddit` with the text as query, search across all Tier 1+2 subs, sort `relevance`, time `month`

### Tier definitions

**Tier 1 (always search in broad mode):** Adsense, Blogging, SEO, webdev, SaaS

**Tier 2 (search when relevant):** Fiverr, Freelancers, juststart, programming, vibecoding, ArtificialInteligence, ExperiencedDevs, Entrepreneur, EntrepreneurRideAlong

### Engagement ranking

Filter: score >= 5 AND comments >= 3 (skip dead threads)

Score formula:
```
engagement_score = (upvotes * 0.4) + (num_comments * 0.6)
```

Recency bonus: posts < 48 hours old get 1.5x multiplier

### Deep dive on a topic

Use `get_post_details` on the chosen post. Extract:
- Key claims and specific numbers from OP
- Top comment insights (often more valuable than OP)
- URLs mentioned (for Playwright crawling)
- Pain points and questions asked
- The contrarian angle or surprising insight

## 3. Fact-Checking Workflow

After extracting claims from Reddit:

1. **WebSearch** — run 2-3 targeted queries to verify key claims
   - Policy changes: "Google AdSense policy change [month] [year]"
   - Stats: verify specific numbers cited in threads
   - Trends: confirm with multiple sources

2. **Playwright** (only when URLs are present in thread)
   - Crawl linked pages for additional data/context
   - Screenshot if useful for the post
   - Extract specific stats, quotes, or evidence

3. **Flag unverifiable claims** — if a claim can't be verified, either drop it or frame it as "Reddit users report X" rather than stating it as fact

## 4. Viral Content Patterns

These patterns are derived from analysis of all Webmatrices posts (REPORT.md) and bishwasbhn's Reddit performance (3,836 karma, top posts 600-1500+ upvotes).

### What works

**Title rules:**
- Jagged, specific numbers: $218, 12x, 3,800, 90% — NOT round numbers (10 tips, 5 ways)
- Contrarian framing: "stop wasting", "won't take your job", "is over?"
- Personal financial stakes: "$1.5K to $218", "RPM down 70%"

**Opening line:**
- State a fact or create tension immediately
- "so that happened." / "25-35% of new code is now AI-assisted."
- NEVER open with context-setting ("I've been working on X for Y months...")

**Body structure:**
- Hook with specific number or uncomfortable stat
- Personal angle — skin in the game
- Data/evidence — tables, real quotes, blockquotes from real people
- Contrarian insight — something unexpected
- Closing question — experts want to answer it

**Voice:**
- Lowercase everything (titles, sentences)
- First person: "i looked at your site", "i've seen this before"
- Short paragraphs, no walls of text
- Slight frustration or urgency: "nobody tells you this but..."
- HTML format for Webmatrices posts: `<p>`, `<strong>`, `<ol>`, `<ul>`, `<code>`, `<a>` tags

**Closing question patterns that drive comments:**
- "anyone else seeing this?" — invites shared experience
- "what are you doing about it?" — invites tactical replies
- "am i wrong here?" — invites debate

### What doesn't work

- Generic help requests with no specifics (no URL, no screenshots, no numbers)
- ChatGPT voice: "I'd be happy to help!", "Great question!", "Here's a comprehensive guide"
- Title Case Headings
- Numbered advice lists without context ("1. Create quality content 2. Improve SEO")
- Posts without a hook in the first line
- Pure tool/utility posts (high views, zero engagement)
- Round numbers in titles (10 tips, 5 ways, 3 steps)
- Posts that don't end with a question

## 5. Audience Segments (quick reference)

Fetch persona data from `get_self_personas` MCP tool. Match personas to segments based on their `metadata.personaTraits.topics` field.

| Segment | Primary Subs | Persona matching criteria |
|---------|-------------|--------------------------|
| AdSense | Adsense, Blogging, juststart | Personas with AdSense/monetization in their topics |
| Fiverr | Fiverr, Freelancers | Personas with freelancing/Fiverr in their topics |
| SEO | SEO, Blogging | Personas with SEO/backlinks/ranking in their topics |
| Web Dev / AI | webdev, programming, vibecoding | Personas with coding/AI/web dev in their topics |
| Startups / SaaS | SaaS, Entrepreneur | Personas with startup/founder/editorial voice |
