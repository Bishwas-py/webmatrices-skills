---
name: trending
description: Multi-platform content discovery across Reddit, Twitter, Dev.to, Medium, and Google News. Scores ideas by viral potential, checks if Webmatrices already covered them, and suggests which persona should write about each. Use when asked for trending topics, content ideas, what to write about, or what's hot.
disable-model-invocation: true
argument-hint: [topic / "non-ai" / "all"]
---

# Trending

Multi-platform content discovery. Searches Reddit, Twitter, Dev.to, Medium, and Google News in parallel, scores ideas by viral potential, and matches them to the best persona for Webmatrices.

Replaces the old trending-topics skill with broader platform coverage and smarter scoring.

---

## ARGUMENTS

| Input | What it does |
|-------|-------------|
| `[topic]` | Searches all platforms for that specific topic |
| `"non-ai"` | Broad scan but explicitly EXCLUDES AI topics |
| `"all"` | Broad scan across all platforms, all topics |
| No args | Same as "all" |

---

## WORKFLOW

### Step 1: Fetch persona data

Call `get_self_personas` MCP to load all personas with their traits. This is needed for Step 5 (matching ideas to personas).

### Step 2: Search ALL platforms in parallel

Fire all searches simultaneously for speed.

**Reddit** (via `reddit_search`):

For broad scans, search across Tier 1 subs: Adsense, Blogging, SEO, webdev, SaaS
For topic scans, search with the topic keyword across relevant subs.

Filter: score >= 5 AND comments >= 3 (skip dead threads)

**Twitter** (via `twitter_search`):

- Max 1-2 word queries. Twitter search is noisy with long queries.
- For "adsense" topic, search: "adsense"
- For "vibe coding" topic, search: "vibe coding"
- For broad scan, search: "webdev", "SEO", "adsense", "AI tools" (separate queries)

**Dev.to** (via `devto_trending`):

- Fetch trending articles
- For topic scans, also use `devto_search` with the keyword

**Medium** (via `medium_tag_feed`):

- For topic scans: search the relevant tag
- For broad scans: search tags "programming", "seo", "artificial-intelligence", "web-development"

**Google News** (via `news_search`):

- Search for the topic or broad terms like "web development", "SEO 2026", "AI coding tools"
- Great for catching breaking news and announcements

### Step 3: Deduplicate and score

Merge results from all platforms. Deduplicate similar topics.

**Scoring formula:**

```
viral_score = (
  engagement_ratio * 30 +       # upvotes/comments relative to platform norms
  cross_platform_presence * 25 + # appears on 2+ platforms = hot
  oh_shit_factor * 20 +          # "oh shit that's me" recognition factor
  paradox_conflict * 15 +        # contains built-in tension or contradiction
  specific_numbers * 10          # has specific data/numbers (not vague claims)
)
```

**engagement_ratio:** Normalize engagement to platform norms. 500 upvotes on Reddit/r/webdev is different from 50 reactions on Dev.to.

**cross_platform_presence:** If the same topic/theme appears on Reddit AND Twitter AND Dev.to, thats a strong signal. Score: 1 platform = 0, 2 platforms = 0.5, 3+ platforms = 1.0

**oh_shit_factor (0-1):** Would the Webmatrices audience read this title and think "oh shit, that's me"? Rate based on:
- Does it describe a specific, narrow situation? (high)
- Does it threaten their livelihood or identity? (high)
- Is it abstract industry analysis? (low)
- Is it a generic how-to? (low)

**paradox_conflict (0-1):** Does the topic contain built-in tension?
- "75% use it. 40% trust it." (high)
- "Revenue is up. Confidence is down." (high)
- "How to improve your SEO" (zero)

**specific_numbers (0-1):** Does the source material include specific data points that can anchor a post?

### Step 4: Check if Webmatrices already covered it

For each top idea, search Webmatrices using `search_posts_by_content` MCP to check if a similar post already exists.

If already covered:
- Note it in the output: "Similar to existing post: [title]"
- Suggest an angle that DIFFERS from existing coverage
- Or deprioritize the idea

### Step 5: Rank and present top 5

For each of the top 5 ideas, present:

```
TRENDING CONTENT IDEAS
══════════════════════

1. SCORE: 87/100
   TOPIC: AdSense RPM crashes across utility sites after March 2026 core update
   SOURCES: Reddit (r/Adsense, 342 upvotes), Twitter (12 mentions), News (Search Engine Journal)
   CROSS-PLATFORM: 3/5 platforms
   WEBMATRICES CHECK: No existing coverage

   SUGGESTED PERSONA: [best match from MCP] (from get_self_personas)
   WHY: [matched persona's voice] fits the topic, [topic] is in their topics array

   SUGGESTED TAG: google-adsense

   EMOTIONAL HOOK: "my rpm went from $4.80 to $1.90 overnight and i havent changed a single thing"
   WHY IT DRIVES COMMENTS: Everyone with AdSense sites can compare their own numbers.
     Specific personal data invites "same here" responses. The unfairness triggers emotional venting.

───────────────

2. SCORE: 74/100
   TOPIC: Vibe coding burnout -- developers losing problem-solving ability
   SOURCES: Reddit (r/vibecoding, 178 upvotes), Dev.to (trending article), Medium (3 pieces this week)
   CROSS-PLATFORM: 3/5 platforms
   WEBMATRICES CHECK: Similar post exists: "vibe coding made me a worse developer"
   → ANGLE SHIFT: Focus on the WITHDRAWAL symptoms, not the damage. "I tried going back to manual coding for a week. Here's what happened."

   SUGGESTED PERSONA: [best match from MCP] (from get_self_personas)
   WHY: [matched persona's voice and backstory relevance to topic]

   SUGGESTED TAG: programming

   EMOTIONAL HOOK: "i couldnt write a for loop from memory. thats when i knew."
   WHY IT DRIVES COMMENTS: Identity threat. Developers who vibe-code will defensively compare
     their own experience. Developers who dont will feel validated.

───────────────

3. SCORE: 71/100
   ...

───────────────

SKIPPED (already covered):
  - "backlinks are dead for small sites" -- existing post by [persona from DB]
  - "AI tools pricing comparison" -- existing post by [persona from DB]
```

### Step 6: Hand off

After presenting, tell the user:
```
Pick a number to write about, or ask for more results.
When ready: /write [persona] [topic]
```

---

## /trending non-ai

When `non-ai` is the argument, explicitly filter OUT:

- Any topic mentioning: AI, LLM, ChatGPT, Claude, GPT, Gemini, Copilot, machine learning, artificial intelligence, neural network, transformer, AGI, AI coding, vibe coding
- Focus on: AdSense, SEO, web monetization, freelancing, web performance, backlinks, content strategy, site architecture, email marketing, analytics
- This exists because AI content saturation makes non-AI topics underserved and higher-opportunity

---

## PLATFORM-SPECIFIC SEARCH TIPS

**Reddit:** Best for community sentiment, pain points, real stories. Sort by "hot" for trending, "top" for proven engagement.

**Twitter:** Best for hot takes and breaking reactions. Keep queries to 1-2 words max. Noisy with longer queries. Good for catching what people are arguing about RIGHT NOW.

**Dev.to:** Best for developer content trends. Trending articles show what the developer community is reading. Good for technical topic validation.

**Medium:** Best for long-form content trends. Tag feeds show what writers are producing. Good for catching topics before they hit Reddit.

**Google News:** Best for breaking announcements, company actions, policy changes. Good for time-sensitive content.

---

## IMPORTANT

- Always fetch personas from `get_self_personas` MCP, NEVER hardcode persona data
- Twitter queries: max 1-2 words. Longer queries return noise.
- Check Webmatrices coverage before recommending. Dont suggest a topic thats already been covered unless you have a fresh angle.
- The "oh shit that's me" factor is the #1 predictor of viral success on Webmatrices. Prioritize ideas that make the reader feel personally called out.
- When suggesting personas, match against their traits from MCP (topics, voice, backstory relevance, recency of last post)
