---
name: app-ideas
description: Research, validate, and publish app ideas to webmatrices.com/app-ideas. Iterative pain-mining via SuperMCP (Reddit/LinkedIn/Dev.to), parallel scout agents to score pain vs competition, then publish to the strict three-section structure (pure Problem + Why Existing Fails + Economics callout). Use when asked to research a new app idea, validate a hypothesis, enrich an existing idea, or publish app idea pages.
disable-model-invocation: true
argument-hint: [track / publish] [topic / slug]
---

# App Ideas

Research and publish app ideas. The /app-ideas/[slug] page is a **decision tool**, not a forum — every field has a specific job and a tight budget. This skill enforces both the research bar (pain ≥7/10, competition ≤4/10, 12+ pain points, 10+ source posts) AND the publishing discipline (pure problem statement, separate why-fails + economics sections, terse Overview/Verdict/Market Timing).

## Subcommands

| Command | What it does |
|---------|-------------|
| `/app-ideas track [topic]` | Iterative pain-mining + parallel scout agents to validate or kill candidate ideas |
| `/app-ideas publish [slug]` | Push validated research to prod via MCP with the strict three-section structure |

If `track` finds a PURSUE-grade idea, hand off to `publish` with the enriched payload.

---

## ALWAYS CHECK EXISTING FIRST

Before any research or push:

1. Read [registry.md](./registry.md) — canonical state of published, killed, and queued ideas (lives in this skill, survives memory wipes)
2. Call `list_app_ideas` (MCP) to verify the registry against live prod (registry may be stale by a few days)
3. Check user memory `project_app_idea_enrichment.md` if present — it's a derived view of the registry, may have session notes
4. If your candidate matches a KILLED hypothesis in the registry → skip Round 1, do not re-research
5. If your candidate matches an existing published slug → use `update_app_idea` / `add_*` tools, never `create_app_idea` (destructive)

The `create_app_idea` MCP tool **deletes and recreates ALL relations** (pain points, source posts, solutions, APIs) on every call. Only safe for first-time creation. After that, always use granular tools.

After every `track` or `publish` session, update both [registry.md](./registry.md) AND `project_app_idea_enrichment.md` so future sessions inherit the state.

---

## /app-ideas track — Research workflow

### Round 1: Parallel scout (kill the duds fast)

For 3-6 candidate ideas, dispatch parallel scout agents. Each agent gets:

- The candidate idea + a working description
- A `do not duplicate` list of existing slugs
- One round only (no deep enrichment yet)
- Required output: pain score, competition score, top 5 source URLs, top 5 pain quotes, verdict (PURSUE / KILL / NEEDS_DEEPER_LOOK), differentiation angle if PURSUE

Each scout uses:

- **Reddit** via `mcp__supermcp__reddit_search` and `mcp__supermcp__reddit_search_subreddit`. Seed queries hunt frustration: `"i am tired of"`, `"i hate having to"`, `"manually i always have to"`, `"there has to be a better way"`, `"wish there was a tool"`, `"spent hours doing"`.
- **LinkedIn** via `mcp__supermcp__linkedin_search`
- **Dev.to** via `mcp__supermcp__devto_search`
- **Comments** via `mcp__supermcp__reddit_get_post(post_id=...)` for top hits — that's where the real pain lives
- **WebSearch** to find competitors with pricing

### Round 1 quality bar

- **PURSUE** only if pain ≥7/10 AND competition ≤4/10 AND audience is specific
- **KILL** if: saturated (5+ funded incumbents at every price tier), wrong-shape product, audience too small or won't pay, or factual hypothesis was wrong
- Be ruthless. 5/5 KILL is a real outcome — better than wasting Round 2 budget

### Round 2: Enrichment (only for PURSUE-grade)

For each PURSUE idea, dispatch ONE enrichment agent to deepen to publication bar:

- 12+ verified pain points (real Reddit/LinkedIn URLs, real upvote counts)
- 10+ source posts (with score, num_comments, snippet, pain_pattern, relevance)
- 4-6 existing solutions with pricing, limitation, highlights, and 1-10 scores (ease, features, flexibility, price, scale, support)
- 4+ API requirements (api_name, purpose, complexity)
- Drafts of all narrative fields per the publish discipline below

### Anti-fabrication rules (CRITICAL)

- Real URLs only. If unsure, call `reddit_get_post` to verify.
- Watch tool_uses on the agent's return — `tool_uses: 0` means the agent fabricated. Reject and re-dispatch with stricter prompt.
- Source diversity: ≤4 pain points from any single thread.
- Recency: ≥4 pain points from posts dated in the last 18 months.
- Pattern field on each pain point: short categorization (2-4 words), e.g. "deadline panic", "spreadsheet hell", "widget distrust".

---

## /app-ideas publish — Publish discipline

The page renders these sections in order. Field budgets are enforced by the page UX, not the schema — keep things terse.

### Field-by-field budget + role (calibrated against 24 shipped pages)

Char/word ranges are derived from actual prod content. The "min" of each range is what the leanest live page uses; the "max" is the upper bound where the section starts feeling bloated. **Median** = where most well-written pages land.

| Field | Type | Chars (min/med/max) | Role — what this field communicates |
|---|---|---|---|
| `description` (Overview) | plain | 110 / 250 / 350 | **What the product does, in one breath.** Product-tense ("Tool that analyzes X", "Phone-first estimator that..."). Reader should know in 2 seconds what they're buying. |
| `target_audience` | plain | 30 / 55 / 100 | **Who specifically.** Role + segment + size hint. Singular focus — not a taxonomy of sub-personas. |
| `revenue_model` | plain | 8 / 20 / 40 | **How money flows in.** Model name ("Monthly subscription", "Freemium", "Per-report + monthly"). Hybrid models are fine if literally accurate. |
| `price_range` | VarChar(100) | 8 / 15 / 50 | **Actual prices.** Always include numbers. "$19-39/mo" or "Free up to 5, $9/mo unlimited". Hard cap 100 chars (DB column limit). |
| `verdict` | plain | 80 / 145 / 250 | **The honest call.** Optional state keyword prefix: `VALIDATED` / `DISMISSED` / no prefix (default Build-tone). The UI styles `VALIDATED` and `DISMISSED` distinctively. |
| `market_timing` | plain | 60 / 130 / 250 | **Why now.** 1–3 sentences naming the technical or market shift (new regulation, AI capability unlocked, demographic curve). Time-bound — answers "why didn't this exist 2 years ago." |
| `competition_notes` | plain | 100 / 170 / 300 | **Named incumbents + the empty slot.** 3–5 short sentences. Each major competitor on its own with pricing + why-it-doesn't-fit-our-ICP. Ends with "the X tier is empty" or equivalent. |
| `tried_and_rejected` | plain (optional) | 50 / 100 / 200 | **What users do today and why it breaks.** Scannable list-prose, comma-separated workarounds. Optional — populated on ~2/3 of pages. |
| `highlights` | string[] | 50–120/bullet | **The case for Build, scannable.** 5–7 one-line bullets. Each bullet = one buying argument, named with a noun (incumbent, regulation, deadline, gap). Avg shipped page = 5.6 bullets. |
| `problem_statement` | rich HTML | 600 / 800 / 1,200 | **PURE pain narrative.** 3–5 short paragraphs. Failure mode + what it looks like in practice + the most-repeated user ask. **NO money figures. NO competitor names.** Both go in their own fields. |
| `why_existing_fails` | rich HTML | 600 / 900 / 1,200 | **Per-competitor breakdown.** Use `<h3>` per competitor or category. Each section: name + pricing in parens, then the specific reason it doesn't solve the user's problem. NEW field (older ideas don't have it). |
| `economics_context` | rich HTML | 500 / 650 / 900 | **Money + market data — visual stat block.** 1–2 framing paragraphs + a `<ul>` of bolded numbers (settlements, market size, pricing pressure, TAM). Renders as emerald callout. NEW field (older ideas don't have it). |

> **Heads up on older ideas:** the 24 ideas published before 2026-04-29 don't have `why_existing_fails` or `economics_context` — they predate the three-section split. Their `problem_statement` is correspondingly bloated (median 1,431 chars vs. 800 for the new structure). When backfilling old ideas, audit `problem_statement` and move money/competitor content out into the new fields.

### Verdict state keywords

The UI checks `verdict?.startsWith('VALIDATED')` and `verdict?.startsWith('DISMISSED')` to apply distinct styling. Use these prefixes when relevant:

- `VALIDATED — ...` — someone is already running this profitably (we've seen the case study)
- `DISMISSED — ...` — we evaluated and killed it (use sparingly; usually `delete_app_idea` is better)
- No prefix — default Build-tone verdict ("Strong build", "Build w/ caveat", etc.)

### Where length goes wrong

- **description > 350 chars** → it's not an Overview anymore, it's a paragraph. Move detail into `problem_statement` or `economics_context`.
- **target_audience > 100 chars** → sub-segments are leaking in. One sentence, not a taxonomy.
- **competition_notes > 300 chars** → split structured competitor analysis into `why_existing_fails` (rich HTML), keep `competition_notes` as a 3–5 sentence scan.
- **problem_statement > 1,200 chars on a NEW idea** → money or competitors are leaking in. Audit and move them to the dedicated fields.
- **problem_statement < 600 chars** → too thin. Add the "what this looks like in real life" middle paragraph.
- **highlights < 5 bullets** → page feels half-baked. Always 5–7.
- **price_range > 100 chars** → Prisma rejects the write. Compress aggressively.

### The three-section split (CRITICAL)

The single biggest publishing rule: **`problem_statement` is a STATEMENT of pain. Money goes in `economics_context`. Competitor analysis goes in `why_existing_fails`.**

- ❌ Do NOT put "$5K-$30K settlements" in problem_statement
- ❌ Do NOT put "TurboBid is $700/yr" in problem_statement
- ✅ Problem statement = the human pain, the failure mode, the most-repeated ask
- ✅ Economics = numbers that frame market size and willingness-to-pay
- ✅ Why existing fails = each competitor and the specific reason it doesn't solve it

### HTML supported in problem_statement / why_existing_fails / economics_context

Page CSS styles: `<p>`, `<h3>`, `<h4>`, `<ul>/<li>`, `<blockquote>`, `<strong>`, `<em>`. Always start with `<p>`.

### Data ownership (softer than the /write skill)

App idea pages are explicitly a research database, so citing Reddit is more legitimate here than in a /write post. But the body narrative should still sound like the curator's analysis, not a Reddit re-post. Calibrate.

**Two attribution channels work in your favor:**

1. **Pain points + source posts** — already explicitly attributed (community badge, source link, upvote/comment count). The reader knows where the data came from. You don't need to repeat it in prose.
2. **Body narrative** (`problem_statement`, `why_existing_fails`, `economics_context`, `verdict`) — should read as the curator's synthesis. Quotes embedded inside owned framing are fine; raw "from r/X" attributions are not.

**The subtle-is-fine rule:**

- ✅ At most **one** explicit Reddit citation per long-form section. E.g. "the same ask resurfaced on r/nonprofit for fourteen years" — frames recency, doesn't source-launder.
- ✅ Quote pulls inside owned narrative: `<p>The honest version: <em>"all the software programs I've seen cost a boatload of money so I've just been doing it my way."</em></p>` — no "from r/electricians" needed; the source is already in the Source Evidence section.
- ✅ "Reddit threads in 2025-2026 are dominated by burnout..." — abstract reference to recency/temperature, fine.
- ❌ "From r/SaaS: ..." then "From r/smallbusiness: ..." then "From r/Entrepreneur: ..." in the same section. Reads like a clipping service, not analysis.
- ❌ "A Reddit user reported X" — passive aggregator energy. Make it the curator's claim, e.g. "owners describe X."
- ❌ Naming subreddits more than 1–2 times across the whole page narrative. The reader gets it.

**Heuristic:** if you removed every subreddit name from the narrative, would the page still feel grounded? If yes, you're fine. If the narrative collapses into vague hand-waving, the citations are doing too much work and the analysis is too thin.

The same /write skill `OWNERSHIP PRINCIPLE` applies to insights: persona owns the framing. The difference here is that source attribution within an explicit research context (with dedicated Source Evidence + Pain Point cards) is allowed in moderation.

### Source community values

Pass **bare community names** in `source_community` — never with the prefix:

- ✅ `"smallbusiness"`, `"electricians"`, `"Baking"`
- ❌ `"r/smallbusiness"`, `"@username"`

The MCP `normalizeCommunity` helper strips `r/` and `@` prefixes on write, but pass clean values to be safe. The UI prepends the prefix per platform — passing `r/X` produces the dreaded `r/r/X` bug.

### Pain point scores

Every pain point should carry the source post's real upvote count + comment count. Don't leave score=0 unless the source post genuinely has no engagement. Map each quote back to its source URL and copy the numbers.

### Per-page minimum bar

For an idea page to look credible publicly:

- 12+ pain points
- 10+ source posts
- 3+ existing solutions with pricing + scored 1-10
- 3+ API requirements
- All narrative fields populated, including `why_existing_fails` and `economics_context`

---

## MCP tools — when to use what

### First-time creation only

`create_app_idea` — full payload, creates the AppIdea + all relations. **Destructive on second call** (deletes and recreates all relations). After first call, never use again on the same slug.

### Editing existing ideas (use these — never re-call create)

- `update_app_idea(slug, ...fields)` — narrative-only update. Pass only the fields you want to change. Doesn't touch relations.
- `add_pain_points(slug, [...])` — append. Does NOT dedupe — only add ones that aren't already there. Check via `get_app_idea` first.
- `add_source_posts(slug, [...])` — append. Dedupes by URL — safe to pass everything.
- `add_existing_solutions(slug, [...])` — append. Auto-increments slug to avoid collisions.
- `add_api_requirements(slug, [...])` — append. Does NOT dedupe — be careful.
- `update_pain_point(id, ...)` / `delete_pain_point(id)` — surgical row edit/delete
- `update_source_post(id, ...)` / `delete_source_post(id)` — surgical row edit/delete
- `update_existing_solution(id, ...)` / `delete_existing_solution(id)` — surgical row edit/delete
- `delete_api_requirement(id)` — surgical delete

To get IDs for surgical updates: call `get_app_idea(slug)` (full data) or query Prisma directly for bulk operations.

### Reading

- `list_app_ideas` — paginated list with scores and counts (no IDs in text output)
- `get_app_idea(slug)` — full data including all relations

---

## Workflow chain

```
/app-ideas track [topic]
    → list_app_ideas (skip duplicates)
    → memory check (skip KILLED ideas)
    → 5 parallel scout agents (Round 1)
    → KILL the duds, return PURSUE list
    → 1 enrichment agent per PURSUE (Round 2)
    → return ready-to-publish JSON payloads

/app-ideas publish [slug]
    → if new: create_app_idea once with full payload
    → if existing: update_app_idea (narrative) + add_*/update_* (relations)
    → verify with get_app_idea
    → save shipped slugs + KILLED slugs to memory
```

---

## State hygiene

After every `track` or `publish` session, update [registry.md](./registry.md) AND mirror to user memory `project_app_idea_enrichment.md`:

- Newly published slugs → add to "Published — credible" table with scores
- Newly killed hypotheses → add to "KILLED hypotheses" table with one-line reason
- Adjacent wedges surfaced during scouting → add to "Adjacent wedges" table
- Thin/saturated published ideas → move to "Published — thin" section, consider `delete_app_idea`
- Bump "Last verified" date in registry.md

The registry is the source of truth; memory is a derived view. If memory gets wiped, the skill still has the full state.

---

## Common failure modes (from prior sessions)

1. **Calling `create_app_idea` to "update" an existing idea** — wipes pain points, source posts, solutions, APIs. ALWAYS use `update_app_idea` + `add_*` after first creation.
2. **Fabricated pain points** — when an enrichment agent returns `tool_uses: 0`, it made up the URLs. Always verify and re-dispatch.
3. **`r/r/SaaS` rendering** — passing `"r/SaaS"` as source_community. Pass bare `"SaaS"`.
4. **Wall-of-text problem_statement** — money + competitors stuffed into Problem section. Split into the 3 dedicated fields.
5. **price_range >100 chars** — fails Prisma VarChar(100). Keep terse.
6. **Pain points showing 0 upvotes / 0 comments** — forgot to populate `score` and `num_comments`. Map each pain quote back to its source post's real numbers.
