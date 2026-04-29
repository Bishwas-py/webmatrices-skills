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

### Reader's decision journey (purpose of each field)

The /app-ideas/[slug] page is a **decision tool**. The reader is asking: "Should I build this?" Every field exists to answer one question in that journey — in order:

| Order | Section | Field(s) | Question it answers | Purpose in the page |
|---|---|---|---|---|
| 1 | The Problem | `problem_statement` | "What pain are we solving?" | **Hook.** Make the pain visceral. Reader should feel it in their chest before they see numbers or competitors. |
| 2 | By the Numbers | `economics_context` | "Is this a real market?" | **Quantify.** Emerald visual block. Stat-stack of settlements / TAM / pricing pressure. Proves the money is there. |
| 3 | Why Existing Solutions Don't Solve It | `why_existing_fails` | "What about [Competitor X]?" | **Eliminate alternatives.** Per-competitor section showing why each named incumbent doesn't fit our ICP. Closes the "but isn't this already done?" objection. |
| 4 | Overview | `description` + `target_audience` + `revenue_model` + `price_range` + `build_complexity` + `competition_level` | "What is the product, who buys it, how much, how hard to build?" | **Synthesize.** The product specs in one card. Reader skim-reads to confirm the shape of the bet. |
| 5 | Verdict | `verdict` + `market_timing` | "Should I actually build this, and why now?" | **Judgment.** The honest call (Build / VALIDATED / DISMISSED) plus the time-bound rationale. The "go/no-go" beat. |
| 6 | What's Been Tried | `tried_and_rejected` | "Aren't users already solving this somehow?" | **Reinforce the gap.** Amber callout. Lists the workarounds (Excel, ChatGPT, calling the supply house) and why each one breaks. |
| 7 | Highlights | `highlights` | "Give me 5–7 reasons to Build." | **Buying arguments.** Scannable bullets, one buying argument each. The case-for-Build TL;DR. |
| 8 | Pain Points | `painPoints[]` | "What exactly do users say?" | **Evidence (granular).** Per-quote cards with sourceCommunity badge, score/comments, "Me too" pill. Sortable by score. |
| 9 | Source Evidence | `postLinks[]` | "Where did this research come from?" | **Receipts.** Linked source posts (Reddit/HN/LinkedIn) with title, score, comments. Proves the curator actually did the work. |
| 10 | Existing Solutions | `existingSolutions[]` | "What does each competitor look like up close?" | **Compare.** Competitor cards with pricing, limitation, 1–10 scores on 6 axes. Per-card detail page for deep dive. |
| 11 | API Requirements | `apiRequirements[]` | "Can I actually build this?" | **Build spec.** For the engineer evaluating feasibility. Lists APIs/integrations needed with complexity rating. |

The narrative fields (1–7) build the case in a specific order — pain → money → alternatives eliminated → product spec → verdict → workarounds → buying arguments. Don't reorder, don't skip.

The data fields (8–11) are the receipts — granular evidence that the narrative isn't hand-waved.

### Field-by-field budget + role

The **Allowed window** is the enforcement target — write to land inside it, never go past the upper bound. The recent 4 ideas (ADA Litigation Shield, SparkQuote, GrantClock, CottageLegal) are the gold standard for the three-section structure and all sit comfortably inside these windows.

The "Reference" column shows where the recent 4 actually landed and where the older 24 (pre-split, looser discipline) drift — use as a sanity check.

| Field | Type | **Allowed (chars)** | Reference: recent 4 \| older 24 (med) | Role — what this field communicates |
|---|---|---|---|---|
| `description` (Overview) | plain | **150 – 300** (hard cap 350) | 161–198 \| 247 | **What the product does, in one breath.** Product-tense ("Tool that analyzes X", "Phone-first estimator that..."). Reader knows in 2 seconds what they're buying. |
| `target_audience` | plain | **50 – 100** | 64–87 \| 53 | **Who specifically.** Role + segment + size hint. Singular focus — not a taxonomy of sub-personas. |
| `revenue_model` | plain | **8 – 30** | 8–25 \| 20 | **How money flows in.** Model name ("Monthly subscription", "Freemium", "Per-report + monthly"). Hybrid OK if literally accurate. |
| `price_range` | VarChar(100) | **8 – 50** (hard cap 100, DB) | 9–36 \| 13 | **Actual prices.** Always include numbers. "$19-39/mo" or "Free up to 5, $9/mo unlimited". |
| `verdict` | plain | **100 – 220** | 135–195 \| 147 | **The honest call.** Optional state prefix: `VALIDATED` / `DISMISSED` / no prefix (default Build-tone). UI styles `VALIDATED` and `DISMISSED` distinctively. |
| `market_timing` | plain | **120 – 260** | 166–251 \| 130 | **Why now.** 1–2 sentences naming the technical/market shift (new regulation, AI unlocked, demographic curve). Time-bound — answers "why didn't this exist 2 years ago." |
| `competition_notes` | plain | **150 – 280** | 178–267 \| 169 | **Named incumbents + the empty slot.** 3–5 short sentences. Each competitor with pricing + why-it-doesn't-fit-our-ICP. Ends with "the X tier is empty." |
| `tried_and_rejected` | plain (optional) | **150 – 450** | 276–422 \| 97 | **What users do today and why it breaks.** Scannable list-prose, comma-separated workarounds. Strongly recommended (recent 4 all populate this). |
| `highlights` | string[] | **5 – 7 bullets**, 50–120 chars each | 6–7 \| 5.6 | **The case for Build, scannable.** Each bullet = one buying argument named with a noun (incumbent, regulation, deadline, gap). |
| `problem_statement` | rich HTML | **600 – 900** (hard cap 1,100) | 683–852 \| 1,431 | **PURE pain narrative.** 3–5 short paragraphs. Failure mode + what it looks like in practice + the most-repeated user ask. **NO money figures. NO competitor names.** Older ideas exceed this because they predate the three-section split. |
| `why_existing_fails` | rich HTML | **600 – 1,000** (hard cap 1,200) | 668–1,014 \| (empty) | **Per-competitor breakdown.** Use `<h3>` per competitor or category. Each section: name + pricing in parens, then the specific reason it doesn't solve the user's problem. NEW field — backfill on older ideas. |
| `economics_context` | rich HTML | **500 – 750** (hard cap 900) | 593–696 \| (empty) | **Money + market data — visual stat block.** 1–2 framing paragraphs + a `<ul>` of bolded numbers (settlements, market size, pricing pressure, TAM). Renders as emerald callout. NEW field — backfill on older ideas. |

> **Read this if backfilling an older idea:** `whyExistingFails` and `economicsContext` are empty for all 24 pre-2026-04-29 ideas. Their `problem_statement` is bloated by ~80% (median 1,431 vs. 800 chars target) because money + competitor analysis was crammed in. Audit, then move that content out into the dedicated fields. Then trim `problem_statement` back into the 600–900 window.

> **The 4-app-idea workflow rule:** when in doubt about a field, check what the recent 4 did. If your draft is wider than their range, trim. The recent 4 are the discipline; the older 24 are historical content that hasn't been migrated yet.

### Verdict state keywords

The UI checks `verdict?.startsWith('VALIDATED')` and `verdict?.startsWith('DISMISSED')` to apply distinct styling. Use these prefixes when relevant:

- `VALIDATED — ...` — someone is already running this profitably (we've seen the case study)
- `DISMISSED — ...` — we evaluated and killed it (use sparingly; usually `delete_app_idea` is better)
- No prefix — default Build-tone verdict ("Strong build", "Build w/ caveat", etc.)

### Where length goes wrong (enforce these caps)

| Symptom | Fix |
|---|---|
| `description` > 350 chars | It's a paragraph, not an Overview. Move detail to `problem_statement` or `economics_context`. |
| `target_audience` > 100 chars | Sub-segments are leaking in. One sentence, not a taxonomy. |
| `revenue_model` > 30 chars | Probably a description, not a model name. Pull out to `description` or `verdict`. |
| `price_range` > 100 chars | Prisma rejects the write (DB hard cap). Compress aggressively. |
| `verdict` > 220 chars | The case-for-Build is leaking in. Move bullets to `highlights`. |
| `market_timing` > 260 chars | Probably listing market data. Move numbers to `economics_context`. |
| `competition_notes` > 280 chars | Structured competitor analysis is leaking. Move to `why_existing_fails` (rich HTML), keep `competition_notes` as a 3–5 sentence scan. |
| `problem_statement` > 1,100 chars on a NEW idea | Money or competitors are leaking in. Audit and move them to `economics_context` / `why_existing_fails`. |
| `problem_statement` < 600 chars | Too thin. Add the "what this looks like in real life" middle paragraph + the most-repeated user ask. |
| `why_existing_fails` > 1,200 chars | Per-competitor section is verbose. Trim each section to name + pricing + 1–2 sentence why-it-fails. |
| `economics_context` > 900 chars | Visual block becomes a wall of text. Cut to ≤6 bullets + 1 framing paragraph. |
| `highlights` < 5 bullets | Page feels half-baked. Always 5–7. |
| `highlights` bullet > 120 chars | Bullet is a paragraph. One buying argument per bullet, one line each. |
| `tried_and_rejected` not populated | Recent 4 all populate this. Add it — it sells the "current world is broken" narrative. |

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
