---
name: app-ideas
description: Research, validate, publish, and audit app ideas on webmatrices.com/app-ideas. Iterative pain-mining via SuperMCP, parallel scout agents to score pain vs competition, publish to the strict three-section structure (pure Problem + Why Existing Fails + Economics callout), and smell-test for cross-field coherence (whether claims in the narrative are backed by the page's own pain points / source posts / existingSolutions, whether pain quotes match their sources, plus length and r/r/X bugs). Use when asked to research a new app idea, validate a hypothesis, enrich/publish an idea, or audit a page for incoherence.
disable-model-invocation: true
argument-hint: [track / publish / smell] [topic / slug]
---

# App Ideas

Research and publish app ideas. The /app-ideas/[slug] page is a **decision tool**, not a forum — every field has a specific job and a tight budget. This skill enforces both the research bar (pain ≥7/10, competition ≤4/10, 12+ pain points, 10+ source posts) AND the publishing discipline (pure problem statement, separate why-fails + economics sections, terse Overview/Verdict/Market Timing).

## Subcommands

| Command | What it does |
|---------|-------------|
| `/app-ideas track [topic]` | Iterative pain-mining + parallel scout agents to validate or kill candidate ideas |
| `/app-ideas publish [slug]` | Push validated research to prod via MCP with the strict three-section structure |
| `/app-ideas smell [slug]` | Audit a published idea for **cross-field coherence** — claims in the narrative backed by pain points / source posts / existingSolutions, pain quotes matching their source URLs, competitor mentions matching solution rows. Plus length violations, r/r/X bugs, and missing data. Read-only. |

If `track` finds a PURSUE-grade idea, hand off to `publish` with the enriched payload. After any `publish`, run `smell` on the slug to verify nothing slipped past discipline.

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

## /app-ideas smell — Audit a published idea

Read-only **coherence** scanner. The main job is checking whether the page's data points hang together — whether claims in the narrative are backed by the page's own pain points / source posts / existingSolutions, whether pain quotes match their source URLs, whether competitor mentions match solution rows. Plus the standard hygiene checks (length budgets, r/r/X bugs, missing scores).

This is **not** an AI-detection scanner like `/webmatrices:smell` — for deep authenticity analysis, run that one on the rendered text. App-ideas smell is about the data hanging together correctly.

### Input

| Input | What it does |
|---|---|
| `[slug]` | Audit one idea via `get_app_idea(slug)` |
| `all` or no args | Audit all ideas via `list_app_ideas` then iterate |
| `recent N` | Audit the N most recent ideas by createdAt |

### Four smell categories

#### Category 1: FIELD DISCIPLINE

Field-level violations against the Allowed budget table above.

| Signal | Severity | Detection |
|---|---|---|
| `description` > 350 chars | HIGH | It's a paragraph, not an Overview. Move detail to `problem_statement` or `economics_context`. |
| `description` < 100 chars | MEDIUM | Too thin, reader can't parse the product shape. |
| `target_audience` > 100 chars | MEDIUM | Sub-segments leaking — should be one sentence. |
| `revenue_model` > 30 chars | LOW | Probably a description; pull out to `description`. |
| `price_range` > 100 chars | HIGH | Prisma will reject the next write. Compress now. |
| `price_range` lacks digits | MEDIUM | "Subscription" without a number is useless to the reader. |
| `verdict` > 220 chars | LOW | Buying arguments are leaking — move to `highlights`. |
| `market_timing` > 260 chars | LOW | Probably listing market data; move numbers to `economics_context`. |
| `competition_notes` > 280 chars | MEDIUM | Structured competitor analysis is leaking; move to `why_existing_fails` (rich HTML). |
| `problem_statement` > 1,100 chars on a NEW idea | HIGH | Money or competitors are leaking in. Audit and move them. |
| `problem_statement` < 600 chars | MEDIUM | Too thin; add the "what this looks like in real life" middle paragraph. |
| `problem_statement` contains `$` | HIGH | Money figures should live in `economics_context`. |
| `problem_statement` mentions any solution name listed in `existingSolutions[].name` | HIGH | Competitor names should live in `why_existing_fails`. |
| `why_existing_fails` is null or empty AND idea created after 2026-04-29 | HIGH | New ideas must populate the three-section split. |
| `economics_context` is null or empty AND idea created after 2026-04-29 | HIGH | Same — new ideas need the visual stat block. |
| `why_existing_fails` > 1,200 chars | LOW | Sections are verbose; trim each to name + pricing + 1-2 sentence why. |
| `economics_context` > 900 chars | LOW | Visual block becomes a wall; cut to ≤6 bullets + 1 framing paragraph. |
| `highlights` < 5 bullets | MEDIUM | Page feels half-baked. Always 5-7. |
| `highlights` bullet > 120 chars | LOW | Bullet is a paragraph; one buying argument per bullet. |
| `tried_and_rejected` is null | LOW | Recent 4 all populate this — sells the "current world is broken" narrative. |

#### Category 2: COHERENCE & EVIDENCE (the main job)

Whether the page's claims hang together. Every number, competitor name, and quote in the narrative should be backed by data on the same page (a pain point, source post, or existingSolution row). Orphan claims = the page reads disconnected.

**Narrative ↔ Page data:**

| Signal | Severity | Detection |
|---|---|---|
| `$` amount appears in `problem_statement` / `economics_context` / `highlights` / `verdict` but no pain point quote, source post snippet, or solution `pricing` field on the page contains the same figure | HIGH | Orphan number. Either back it with evidence or drop it. (Externally well-known facts like "FTC v. accessiBe $1M" are exempt — citation is the source.) |
| Competitor name appears in `competition_notes` / `why_existing_fails` / `verdict` / `tried_and_rejected` but no matching entry in `existingSolutions[]` (case-insensitive name match) | HIGH | Solution should be added to `existingSolutions` so readers can see scores + pricing on its card. |
| `existingSolutions[]` row whose `name` is never referenced in any narrative field | MEDIUM | Why is it there? Either add a sentence about it in `competition_notes` / `why_existing_fails`, or delete the row. |
| Solution `pricing` mentioned in narrative differs from the row's `pricing` field (e.g. narrative says "$199/mo" but solution row says "$299/mo") | HIGH | Stale or contradictory data — pick one source of truth. |
| `target_audience` claims a specific role/billing rate/segment size that doesn't appear in any pain point or source post snippet | MEDIUM | Audience is hand-wavy — back it with at least one evidence quote. |
| `verdict` claims urgency/window not reinforced by `market_timing` or `economics_context` | MEDIUM | The "why now" should appear in at least 2 places consistently. |
| `highlights` bullet asserts a fact (number, regulation date, competitor outcome) not present anywhere else on the page | MEDIUM | Orphan bullet. Either add the supporting context elsewhere or rewrite the bullet to reflect what's actually on the page. |
| `whyExistingFails` contains an `<h3>` for a competitor not in `existingSolutions[]` | HIGH | Section heads must align with the cards rendered below — readers expect to scroll to the matching card. |

**Pain point ↔ Source post:**

| Signal | Severity | Detection |
|---|---|---|
| Pain point's `sourceUrl` is not the URL of any row in `postLinks[]` for the same idea | HIGH | The receipt isn't on the page. Either add the source post via `add_source_posts`, or fix the pain point's URL. |
| Pain point `sourceCommunity` doesn't match the subreddit in its `sourceUrl` (e.g. URL `/r/Baking/comments/...` but community `"Cooking"`) | HIGH | Mis-tagged. Update via `update_pain_point`. |
| Pain point `score` / `num_comments` differ from the matching source post's numbers (when both exist on the page) | HIGH | Same Reddit thread = one data point — values must agree. Copy from the source post. |
| Pain point quote isn't a verbatim or near-verbatim string from the linked Reddit thread (verify via `reddit_get_post`) | HIGH | Paraphrased or fabricated. Replace with the actual quote. |

**Source post ↔ URL:**

| Signal | Severity | Detection |
|---|---|---|
| Source post `sourceCommunity` doesn't match URL path subreddit | HIGH | URL `/r/SaaS/comments/...` with `sourceCommunity="smallbusiness"` is wrong. |
| Source post `sourceType=reddit` but `url` isn't a `reddit.com/r/...` form | HIGH | Wrong sourceType or wrong URL. |
| Source post `score` is order-of-magnitude off from a freshly-fetched value (verify via `reddit_get_post`) | LOW | Reddit scores drift — only flag if numbers differ by >10x. |

**Semantic / theme coherence (the deep checks):**

These are about whether the narrative's *meaning* aligns with what the page's data actually shows. Use fuzzy matching where exact comparison isn't possible.

| Signal | Severity | Detection |
|---|---|---|
| Quoted text inside `<em>` or `<blockquote>` in `problem_statement` / `why_existing_fails` / `economics_context` doesn't appear (verbatim or ≥80% string overlap) in any `painPoint.quote` on the page | HIGH | The narrative is "quoting" something the page doesn't actually evidence. Either add the matching pain point, or rewrite the quote to one that exists. |
| Pain point `pattern` values (the 2–4 word categories) are not reflected in `problem_statement` text by at least their root keywords | MEDIUM | Page has 15 pain points tagged "deadline panic" / "spreadsheet hell" / "volunteer turnover" but problem statement talks about something else entirely → narrative drift from data. |
| Narrative claims "the most repeated ask is X" / "the most-cited complaint" / "the dominant pattern" but the highest-`score` pain point's content doesn't reflect X | HIGH | The asserted top theme isn't the actual top theme. Re-rank or rewrite. |
| `why_existing_fails` `<h3>` for competitor X has no pain point on the page that mentions X by name (or the underlying complaint about X) | MEDIUM | Section is unsupported — either add a pain point that complains about X, or trim the section to a sentence in `competition_notes`. |
| Bolded number in `economics_context` (e.g. `<strong>$5K-$30K</strong>`) is not traceable to: (a) a `painPoint.quote`, (b) a `sourcePost.snippet`, (c) a `solution.pricing`, (d) an externally-cited fact named in narrative ("FTC v. accessiBe $1M order") | HIGH | Floating number — if it can't be sourced, it's hand-waved. Either back it or drop it. |
| Narrative uses recency markers ("recent", "in 2025", "fresh", "post-2024", "2025-2026 threads") but no source post URL has a Reddit slug consistent with that period (Reddit URL slugs are roughly time-ordered; post IDs starting with `1n...`/`1o...`/`1p...`/`1q...`/`1r...`/`1s...` are 2024-2026) | MEDIUM | Recency claim isn't backed. Either remove the recency framing or add a recent source. |
| Narrative uses frequency markers ("every week", "the same ask resurfaces", "for fourteen years") but source posts cluster in a single time window | MEDIUM | Frequency claim isn't backed by spread of source dates. |
| `problem_statement` declares a count ("three failure modes", "five repeated complaints") but the structured page doesn't have that count (e.g. only 2 distinct pain `pattern` categories present, or only 2 sections in `why_existing_fails`) | MEDIUM | Counted claim doesn't match counted reality. Adjust the number or add the missing items. |
| Pain points have ≥10 entries but ≤2 distinct `pattern` values | MEDIUM | The "wide range of pain" implication doesn't hold — re-pattern the pain points or acknowledge the narrow theme. |
| `verdict` says "MVP in N weeks" or similar effort estimate that contradicts `build_complexity` score (e.g. "4-6 weeks" with `build_complexity=9`) | MEDIUM | Effort claim mismatches scored complexity. Pick one. |
| `target_audience` declares a specific persona but no pain point quote uses that persona's voice or context (e.g. audience says "solo electricians billing $85-150/hr" but no pain quote mentions billing rates, solo work, or electrician-specific context) | MEDIUM | Audience hand-waved — at least one pain point should sound like the named persona. |
| Each `highlights` bullet must have ≥1 supporting field elsewhere on the page (matching pain point, source post snippet, economics fact, solution row, or external citation in narrative) | MEDIUM | Orphan bullet — the case-for-Build is claiming things the page doesn't otherwise show. |

#### Category 3: DATA QUALITY (relations)

Quality of the underlying pain points, source posts, solutions, and APIs.

| Signal | Severity | Detection |
|---|---|---|
| Any pain point with `sourceCommunity` starting with `r/` (sourceType=reddit) | HIGH | The r/r/X rendering bug. UI prepends `r/` — pass bare names. Fix via `update_pain_point`. |
| Any source post with `sourceCommunity` starting with `r/` | HIGH | Same bug. Fix via `update_source_post`. |
| Any pain point with `sourceCommunity` starting with `@` (sourceType=twitter) | HIGH | UI prepends `@` — same r/r/ class of bug. |
| Pain point with `score=0 AND num_comments=0` AND its sourceUrl appears in `postLinks[]` with non-zero numbers | HIGH | Forgot to populate scores. Map quote → source post → copy the real numbers. |
| Pain point missing `pattern` field | HIGH | Required field; renders the category pill empty. |
| Pain point `pattern` > 30 chars | LOW | Should be 2-4 word category, not a sentence. |
| Pain point quote shorter than 30 chars | MEDIUM | Probably truncated or summary, not a real quote. |
| > 4 pain points sharing the same sourceUrl | MEDIUM | Low source diversity — over-sampled one thread. |
| Zero pain points from posts dated within 18 months | MEDIUM | Recency gap; the pain may have changed. |
| Source post with `score=0 AND num_comments=0` | LOW | Either thread is dead or numbers weren't pulled. Verify via `reddit_get_post`. |
| Source post missing `snippet` | LOW | Card looks bare without a one-liner. |
| Source post missing `pain_pattern` | LOW | Used to color/group cards in UI. |
| Existing solution missing `pricing` | MEDIUM | The pricing badge is a key buyer-comparison field. |
| Existing solution missing `limitation` | MEDIUM | Reader can't see why-it-fails without it. |
| Existing solution with all 6 scores at default 5 | MEDIUM | Scores weren't actually evaluated. |
| Existing solution with `description` < 50 chars | LOW | Card body is bare. |
| `pain_points.length` < 12 | HIGH | Below publication credibility bar. |
| `postLinks.length` < 10 | HIGH | Below publication credibility bar. |
| `existingSolutions.length` < 3 | MEDIUM | Below the comparison threshold readers expect. |
| `apiRequirements.length` < 3 | LOW | Build spec is sparse for engineer evaluators. |

#### Category 4: AUTHENTICITY (light — defer to `/webmatrices:smell` for deep analysis)

Quick AI-tell + source-laundering checks on the body narrative. This skill is not the right place for deep authenticity analysis — for that, copy the rendered text and run `/webmatrices:smell "..."`. The checks here catch only the most obvious leaks.

| Signal | Severity | Detection |
|---|---|---|
| Body narrative contains `From r/X:` or `from r/X` | HIGH | Source-laundering. Embed the quote in owned framing instead. |
| Body narrative contains `A Reddit user`, `One commenter`, `Someone on Reddit` | HIGH | Passive aggregator energy. |
| Same subreddit mentioned > 1 time in any single long-form section | LOW | Citation overload — pain point + source post cards already attribute. |
| Body narrative contains banned AI vocabulary: `delve`, `leverage`, `comprehensive`, `robust`, `streamline`, `let that sink in`, `fair point` | HIGH | Reads as AI-generated. |
| `<blockquote>` not wrapped in `<em>` | LOW | Page convention is italic blockquotes. |

For deeper checks (THE THIN LINE tests, hypothetical realness, polite-stranger test, voice consistency), run `/webmatrices:smell` on the rendered page text.

### Verdict-state sanity checks

Compare scores against verdict prefix:

| Signal | Severity | Detection |
|---|---|---|
| `painIntensity ≤ 5 AND competitionLevel ≥ 7 AND verdict does NOT start with DISMISSED` | MEDIUM | Should this be killed? Either re-justify the build or DISMISS / `delete_app_idea`. |
| `painIntensity ≥ 8 AND competitionLevel ≤ 3 AND verdict does NOT mention specific go-build action` | LOW | Strong setup but verdict reads tepid — sharpen the call. |
| `verdict starts with VALIDATED` but no `existingSolutions` entry has the validating case study referenced | MEDIUM | VALIDATED prefix should be backed by a specific competitor making money. |
| `verdict starts with DISMISSED` but the idea is still live (not soft-deleted) | LOW | Consider `delete_app_idea` to keep the public catalog clean. |

### How to run the deep semantic checks

The structural checks (length, r/r/X, missing fields) are mechanical — run them first via raw field comparisons. The semantic coherence checks need more care:

1. **Quote echo** — extract every string inside `<em>`, `<blockquote>`, or quoted text in the narrative HTML fields. For each, normalize (lowercase, strip punctuation/whitespace) and check substring containment against the normalized concatenation of all `painPoint.quote` values. Anything below ~80% match → flag.
2. **Theme overlap** — tokenize all `painPoint.pattern` values into root keywords (e.g. "deadline panic" → `["deadline", "panic"]`). For each pattern, at least one of its root keywords should appear in `problem_statement` text (case-insensitive). Flag patterns with zero keyword presence.
3. **"Most repeated ask" claim** — when narrative contains phrases like "most repeated ask", "dominant pattern", "most common complaint", extract the surrounding clause and compare it against the highest-`score` pain point's quote/pattern.
4. **whyExistingFails ↔ pain points** — for each `<h3>` in `why_existing_fails`, extract the heading text. Check that at least one `painPoint.quote` mentions any word from the heading (the competitor name typically).
5. **Number sourcing** — extract every `<strong>NUMBER</strong>` from `economics_context`. For each, scan all pain quotes, source post snippets, and solution pricing fields for the same figure. If not found, scan the surrounding economics_context paragraph for an external citation pattern ("FTC v.", "DOJ", "EAA", a date, etc.). If still not found → flag.
6. **Recency check** — Reddit post slugs like `/comments/1s305p8/` start with a base-36-ish character that roughly indicates posting time. Posts starting with `1s`/`1q`/`1r` are late 2025; `1n`/`1o`/`1p` are mid-late 2025; older slugs (5-6 chars) are pre-2024. If narrative says "recent" or "2025-2026" but all source URLs have older slugs → flag.
7. **Frequency check** — count distinct `sourcePost.url` paths and look at slug spread. If narrative says "every week" or "for fourteen years" and all 12 source posts cluster in slugs from one quarter → flag.

For the trickier ones (semantic similarity beyond substring matching), an LLM-judge call is acceptable: pass the narrative claim + the page evidence and ask "does the evidence support this claim? yes/no/partial".

### Output format

For each scanned slug, return a structured report:

```
=== <slug> ===
SCORE: <pass/warn/fail>
HIGH: <count>
MEDIUM: <count>
LOW: <count>

[HIGH] description (412 chars > 350): "..." → trim to ≤350, move detail to problem_statement
[HIGH] problem_statement contains "$5K-$30K" — move to economics_context
[HIGH] painPoint cmok... has sourceCommunity="r/SaaS" → update_pain_point with "SaaS"
[MEDIUM] competition_notes (321 chars > 280) → split into why_existing_fails
[LOW] highlights[3] (138 chars > 120) → tighten to one buying argument
...
```

Group findings by severity. Cite the exact id/field/value so the reader can issue the surgical fix via `update_app_idea` / `update_pain_point` / `update_source_post`.

### Auto-fix mode (use sparingly)

For obvious mechanical fixes only — never auto-fix narrative or interpretation:

- ✅ Strip `r/` prefix from all `sourceCommunity` values via `update_pain_point` / `update_source_post` — the normalize helper handles it on write
- ✅ Fix pain point `score`/`num_comments` from matching source post numbers
- ❌ Trim `description` text — needs a human eye for what to cut
- ❌ Move money out of `problem_statement` — interpretation required
- ❌ Add missing `why_existing_fails` content — needs research

When invoking auto-fix, always print the proposed changes first and ask for confirmation.

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
    → registry + memory check (skip KILLED ideas)
    → 5 parallel scout agents (Round 1)
    → KILL the duds, return PURSUE list
    → 1 enrichment agent per PURSUE (Round 2)
    → return ready-to-publish JSON payloads

/app-ideas publish [slug]
    → if new: create_app_idea once with full payload
    → if existing: update_app_idea (narrative) + add_*/update_* (relations)
    → verify with get_app_idea
    → run /app-ideas smell [slug] to catch any leaks before declaring done
    → save shipped slugs + KILLED slugs to registry + memory

/app-ideas smell [slug]
    → get_app_idea(slug) (or list_app_ideas + iterate for "all")
    → run all 3 smell categories (FIELD DISCIPLINE, OWNERSHIP+AUTHENTICITY, DATA QUALITY)
    → return per-slug report grouped by severity (HIGH / MEDIUM / LOW)
    → optionally auto-fix r/r/ bugs and missing pain-point scores (with confirmation)
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
