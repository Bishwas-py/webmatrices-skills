---
name: write-post
description: Write and publish long-form posts as a Webmatrices persona — for Webmatrices forum, Reddit subreddits, HN, or Dev.to. Pulls voice from persona samples, shape from platform/community conventions, perspective from persona backstory. Researches via SuperMCP, fact-checks claims, previews via Playwright, publishes via MCP. Use when asked to write a post, draft an article, publish content, or seed a thread on a subreddit.
disable-model-invocation: true
argument-hint: [persona] [topic]
---

# Write Post

Write a long-form post as a persona. Posts are designed content — unlike replies, they have structure, hooks, and engagement design. The persona's samples drive voice; the target platform's conventions drive shape; the persona's backstory drives perspective. Universal content blocks always apply.

For the universal block list, see [_shared/universal-blocks.md](../_shared/universal-blocks.md).
For engagement psychology, see [_shared/engagement-psychology.md](../_shared/engagement-psychology.md).
For reply quality patterns (when post mode generates a follow-up comment plan), see [_shared/reply-patterns.md](../_shared/reply-patterns.md).
For trending topic discovery, use `/trending` first, then hand off here.
For persona matching, see the [audience-matcher skill](../audience-matcher/SKILL.md).

For short reactions (comments, replies to threads), use `/write-reply` instead.

---

## Subcommands

| Command | What it does |
|---------|-------------|
| `/write-post [persona] [topic]` | Draft a post as that persona about that topic |
| `/write-post preview` | Preview the current draft via `preview_post` MCP + open in Chrome |
| `/write-post publish` | Publish via `create_post` MCP + `generate_og_image` |

If no persona is provided, auto-match using audience-matcher logic after fetching personas from MCP.
If no args at all, ask what to write about.

---

## CORE RULE

> **Voice from samples. Shape from platform conventions. Perspective from backstory. Universal blocks always apply.**

| Layer | Source | Drives |
|-------|--------|--------|
| Voice | `writingSamples` | sentence shape, casing, slang, punctuation rhythm, apostrophe pattern |
| Shape | Target platform conventions | title format, opener style, paragraph rhythm, link density, closing pattern, length norms |
| Perspective | `backstory`, `topics`, `opinionStrength`, `signaturePhrases` | what stance, which experiences surface, which signature phrases appear, why this persona cares |
| Hard limits | [_shared/universal-blocks.md](../_shared/universal-blocks.md) | banned patterns regardless of any of the above |

The old rule "every hypothetical detail must come from backstory" still holds, but only for **perspective** (which details the persona references). It does NOT drive sentence-level voice — that's the samples' job.

---

## PERSONA DATA — ALWAYS FROM MCP

**NEVER hardcode persona usernames, IDs, voice descriptions, or backstories.**

At the start of every `/write-post` invocation:

1. Call `get_self_personas` to fetch all personas with their metadata.
2. Each persona has `id`, `username`, `metadata.personaTraits` (voice, topics, opinion strength, backstory, writing samples, apostrophe patterns).
3. If a persona name is provided, match it against the fetched list.
4. If no persona specified, use traits to auto-match against the topic.

### Voice Continuity

After selecting a persona, fetch their last 2-3 posts via `list_posts` with `authorId` to:

- Match their recent voice (a persona's voice naturally evolves; recent posts are the freshest sample set)
- Avoid repeating topics they recently covered
- Check backstory consistency (don't contradict something they said last week)
- Glance at apostrophe pattern (soft preference; natural drift is fine)

---

## PLATFORM SHAPE

Posts target specific platforms. Each platform has conventions that drive *shape* — title format, opening style, link density, paragraph length, closing pattern. Voice stays sample-driven; shape adapts per platform.

### Webmatrices forum (default)

The default target. Uses Webmatrices's native long-form conventions:

- Title: lowercase, punchy, specific number or built-in tension. No colons. No question marks.
- Opening: hook in first 1-3 sentences. Fact, contrarian take, or specific number. No setup.
- Subheadings: `<h2>` only. Zero emojis in `<h2>`. Lowercase, punchy, specific.
- Body: HTML — `<p>`, `<h2>`, `<strong>`, `<ol>`, `<ul>`, `<blockquote>`, `<a>`.
- Paragraph rhythm: short paragraphs, lots of whitespace. Vary lengths.
- Links: max 3-4 total, max 1 per paragraph, primary sources only (official docs, papers).
- Closing: statement, not engagement-bait question. Quiet confidence.

### Hacker News post

- Title: lowercase, descriptive, specific. No clickbait. No "X is dead" or "5 things you...".
- Body: plain text or minimal formatting. No headers.
- One core idea per paragraph. Compact.
- Links: 2-3 max, all to primary sources.
- Tone: technical, slightly tired, specific.

### Reddit post (subreddit-targeted)

- Title format depends on the subreddit. Check what's pinned, what's recent, what's top this week.
- Body should match what works on the specific sub:
  - r/programming: technical, source-linked, hot-take welcome
  - r/SaaS: numbers-forward, indie-builder voice
  - r/Entrepreneur: longer-form personal narrative
  - r/webdev: technical but practical, no theory
- If the sub is unfamiliar, browse 5 top posts of the past week via `browse_subreddit` MCP and match their shape.

### Dev.to / Medium

- Title: descriptive, ranks well for search. Numbers and "how to" formats work here even though they're banned elsewhere.
- Subheadings encouraged.
- Code blocks for any technical content.
- Cover image expected (note in output; this skill does not generate cover images).

---

## RESEARCH VIA SUPERMCP

Before writing, gather current data using SuperMCP tools. Run in parallel where possible:

| Tool | When to use |
|------|-------------|
| `reddit_search` | Community sentiment, pain points, real stories |
| `twitter_search` | Hot takes, breaking reactions (max 1-2 word queries) |
| `devto_trending` | Developer content trends |
| `medium_tag_feed` | Long-form content trends by topic |
| `news_search` | Current events, announcements, data |

Extract: specific numbers, pain points, contrarian angles, universal frustrations, verifiable claims.

If the target is a specific subreddit, also call `browse_subreddit` for that sub to learn its shape conventions.

---

## OWNERSHIP PRINCIPLE

The persona ALWAYS owns the take. Never:

- "Someone on r/X said..."
- "A thread on HN revealed..."
- "2,462 people upvoted a post saying..."
- "The internet is split on..."

Instead, the persona encountered the idea through their own work. Reddit and HN give you the THEME. The persona provides the STORY. See `_shared/engagement-psychology.md` and the existing `write/skill.md` for the full Ownership Principle and Source Laundering Rule — those sections still apply here.

---

## ENGAGEMENT-READY DESIGN

Posts are designed to attract organic comments. The Gap Principle, Engagement Hooks, Hypothetical Numbers, Hook Engineering, Title Rules, Emotional Pacing, and Engagement Blueprint sections from `skills/write/skill.md` all still apply for post mode. Reference those sections rather than duplicating here.

Key reminders:

- Leave one missing angle for commenters to supply
- Include one debatable claim (correction comments are 3-4x longer than agreement comments)
- Use awkward-specific numbers ($47.23, 1,247 sessions) — never round
- End with a real question the persona wants answered, not generic engagement bait
- Output an engagement blueprint alongside the draft (suggested first/second comments, distribution, timing)

---

## DEEP AUTHENTICITY

Surface tells (em-dashes, banned phrases) are table stakes — those are covered by universal blocks. Deep authenticity is structural. See [smell skill THIN LINE section](../smell/skill.md#thin-line) for the full test set.

Five tests every draft must pass:

1. **SPECIFICITY FROM BACKSTORY.** Hypothetical details must come from the persona's lived experience in `backstory`. Not "a cafe" — the specific cafe. Not "I lost my job" — the specific event from their timeline.
2. **EARNED OPINIONS.** Opinions must trace back to backstory experience, not logical derivation. The persona arrived at the view through life, not reasoning.
3. **EMOTIONAL INVESTMENT MAPPING.** The section the persona cares about is the tightest-written. Setup is looser. Uncertainty is shortest. AI writes everything at even quality; humans don't.
4. **VISIBLE THINKING.** Show the journey: "I thought it was X. Then I ran the numbers. Actually..." Not pre-formed conclusions.
5. **THINGS LEFT UNSAID.** Don't explain what the audience already knows. Trust shared community context.

**The Template Test (final gut check):** Could a different persona have written this with minor voice changes? If yes, kill it. Content must be SPECIFIC to this persona's life.

---

## WORKFLOW

### Step 1: Fetch personas and determine author

1. Call `get_self_personas`.
2. If persona provided, match.
3. If not, auto-match topic against persona traits (topics, voice, backstory relevance).
4. Persona fatigue check: if same persona posted in last 3 days, deprioritize unless topic is a perfect fit.
5. Fetch persona's last 2-3 posts via `list_posts` for voice continuity.

### Step 2: Determine target platform

Default is Webmatrices forum. If the topic or persona suggests a different target (Reddit thread, HN, Dev.to), confirm with user or use platform-shape conventions accordingly.

### Step 3: Research

Run SuperMCP tools in parallel. If target is a specific subreddit, also call `browse_subreddit` to learn the sub's shape.

### Step 4: Write the draft

1. **Title** — apply platform title rules. For Webmatrices: lowercase, no colon, specific number if possible.
2. **Opening line** — hook immediately. Fact or tension. No context-setting.
3. **Body** — sample-driven voice. Platform-driven shape. Backstory-driven specifics.
4. **Closing** — platform-appropriate. For Webmatrices: statement, not bait. For Reddit: depends on sub conventions.
5. **Format** — match platform.
6. **Engagement design** — apply Gap Principle. One uncovered angle. One debatable claim. Real closing question if appropriate.

### Step 5: Fact-check (MANDATORY)

Auto-run `/fact-check` internally on every verifiable claim before presenting. See existing `write/skill.md` fact-check section for the full protocol. Primary sources only.

### Step 6: Universal block check

Run the draft through every block in [_shared/universal-blocks.md](../_shared/universal-blocks.md):

- [ ] No "Honestly," opener
- [ ] No hyphen `-` as clause punctuation
- [ ] No em-dash (—)
- [ ] No formal connectors (`Furthermore`, `Additionally`, `Moreover`, `In conclusion`, `That said,` opener, `Specifically,` opener, `Interestingly`, `Notably`, `It's worth noting`)
- [ ] No "Not X, but Y"
- [ ] No "Not X. Not Y. Z." triple-repetition
- [ ] No AI vocabulary (`comprehensive`, `robust`, `streamline`, `delve`, `leverage`, `crucial`, `game-changer`)
- [ ] No performed emotion
- [ ] No engagement-bait closer ("What do you think?")
- [ ] No "Fair point" / "Great question!" / "Absolutely!" openers

Plus existing post-mode quality checks:

- [ ] `<h2>` subheadings only. Zero emojis in `<h2>`.
- [ ] Max 3-4 links total, max 1 per paragraph.
- [ ] Hypothetical numbers are awkward-specific, not round.
- [ ] Varied section quality (the part the persona cares about is tighter).
- [ ] At least one mid-thought correction or abandoned tangent.
- [ ] Source links present for any technical claim.
- [ ] Source laundering check — no "someone said", no "another commenter", no subreddit name as source.
- [ ] Cross-persona check if multiple personas are engaging on this post.

### Step 7: Present to user

```
PERSONA:        [username] ([why this persona])
TARGET:         [platform / subreddit]
TITLE:          [the title]
TAGS:           [suggested tag slugs, Webmatrices only]
PUBLISH DATE:   [staggered timestamp]
LENGTH:         [N words]
SAMPLES USED:   [first 80 chars of samples that anchored voice]

BODY:
[full HTML or platform-native format]

BLOCK CHECK:    ✓ all clear (or list failures)
QUALITY NOTES:  [any flags from the checklist]

ENGAGEMENT BLUEPRINT:
- Trigger type: [correction / recognition / status-signal / emotional-vent]
- Gap left: [specific angle not covered]
- Debatable claim: [paragraph N]
- Suggested first comment: [persona] [type] (~Xmin after post)
- Suggested second comment: [persona] [type] (~Xhrs after first)
- Suggested tangent: [persona] [adjacent topic] (next day)
- Distribution: 70% agree / 20% pushback / 10% tangent
```

Ask for approval or edits.

### Step 8: Preview (on `/write-post preview`)

1. Call `preview_post` MCP with `title`, `body`, `authorId`, `tagSlugs`
2. Open `http://localhost:8897` in Chrome via Playwright (`browser_navigate`)
3. User reviews visual preview

### Step 9: Publish (on `/write-post publish`)

1. Call `create_post` MCP with `authorId`, `title`, `body`, `tagSlugs`, `createdAt`
2. Call `generate_og_image` MCP with the new post's slug
3. Display result (post ID, slug, URL)

The MCP handles content moderation, slug generation, tag counts, audit logs, email notifications.

### Staggering multiple posts

- 2 posts: 4-8 hours apart
- 3 posts: spread across 2-3 different days
- 4+ posts: spread across 3-5 days, max 2 per day

Pick realistic times (not 3am unless persona is a night owl). Different personas should post at different times.

---

## PROJECT BLOCK (promote intent)

When a PROJECT block is provided:

```
PROJECT:
  name: [project name]
  pitch: [one-line pitch]
  url: [url]
  style: subtle | story | direct | question
```

Mention the project **once**, naturally, in the persona's sample voice. Same rules as `/write-reply`:

- Do not paste pitch verbatim
- Do not gate on relevance
- Style determines weave-in (subtle / story / direct / question)
- Link allowed if natural to the platform and persona

For posts, the project mention is usually inside the body (one paragraph), not the opener and not the closer. Closer should still be a real question or statement.

---

## AVAILABLE TAGS (Webmatrices target)

Use `list_tags` MCP to get current tags. Common ones:

- `google-adsense` — AdSense topics
- `digi-work` — Freelancing/Fiverr/Upwork
- `programming` — Coding/tech
- `ai-founder` — AI tools/startups
- `sveltekit` — SvelteKit framework
- `django` — Django framework

---

## IMPORTANT

- **Voice from samples. Shape from platform. Perspective from backstory.** If output sounds like an essay regardless of persona, the backstory drove voice — re-anchor.
- **Universal blocks override everything.** Even if a persona's samples contain a banned pattern, output strips it.
- **One generation per request.** Generate, run checks, present. Don't silently re-draft. If checks fail, fix once and present. If still failing, tell the user.
- **Auto-run `/fact-check`** on every draft before presenting. Wrong facts kill posts on HN/Reddit.
- **Show the draft for approval** before publishing. Never auto-publish.
- **Engagement blueprint is a suggestion** for `/simulate-engagement`, not a requirement.
- **Stagger multiple posts** across different days and times.
