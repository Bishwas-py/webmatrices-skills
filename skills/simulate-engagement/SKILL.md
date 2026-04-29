---
name: simulate-engagement
description: Orchestrate organic-looking engagement on Webmatrices posts. Fetches engagement personas from MCP, scans posts, picks personas, decides comment types with realistic timing gaps, runs /smell before publishing, previews via Playwright, and outputs an engagement plan for approval before executing. Use when asked to simulate engagement, seed comments, reply to posts, or create organic-looking discussion.
disable-model-invocation: true
argument-hint: [post-slug-or-scope]
---

# Simulate Engagement

Orchestrate organic-looking engagement across Webmatrices posts. This skill replaces manual one-off replies with a strategic engagement system that considers personas, timing, comment types, thread dynamics, and platform phase.

For engagement psychology research, see [engagement-psychology.md](../_shared/engagement-psychology.md).
For reply quality patterns, see [reply-patterns.md](../_shared/reply-patterns.md).
For content patterns, see [content-engine.md](../_shared/content-engine.md).

## Arguments

- `[post-slug]` -- simulate engagement on a specific post
- `[scope: recent / unanswered / all]` -- scan posts matching a scope
- No args -- scan recent posts and suggest which need engagement

---

## PERSONA DATA -- ALWAYS FROM MCP

**NEVER hardcode persona usernames, IDs, voice descriptions, or active hours.**

At the start of every simulate-engagement invocation:

1. Call `get_self_personas` MCP tool to fetch ALL personas
2. Filter for engagement-type personas (those with `type: engagement` in their traits, or all personas if type is not set)
3. Each persona includes `metadata.personaTraits` with: voice, topics, opinion strength, backstory, apostrophe patterns, writing samples
4. Use these traits for voice matching, NOT a static table
5. Fetch each persona's recent activity via `list_posts` and `get_comments` to check activity levels

### Apostrophe Patterns (soft preference)

Apostrophe consistency is a soft preference, not a strict rule. Lean toward the persona's pattern from `metadata.personaTraits.apostrophes` but dont enforce it rigidly. Natural drift is expected — real people are inconsistent with apostrophes.

---

## DEEP PERSONA READING (MANDATORY BEFORE ANY COMMENT)

This is the highest-leverage step. Skipping it produces generic comments that get caught by /smell. Before drafting a single sentence, you must understand each commenting persona at a level where you could answer "what would they NOT say?"

### Per-persona checklist before writing

For EACH persona you plan to use as a commenter:

1. **Read the full backstory block** from `metadata.personaTraits.backstory` — every field. Age, career timeline, location, income, tools used, recent wins, current frustrations, family details if present.
2. **Read all writing samples** from `metadata.personaTraits.writingSamples` (or equivalent). Note their sentence rhythm, paragraph length, signature framings, what they get wrong.
3. **Read the topics list** to see what they actually care about. A persona whose topics include "AdSense rejections" but not "remote work" should not be the one explaining commercial real estate dynamics.
4. **Read their recent posts and comments via `list_posts` and `get_comments`**. What examples have they already used? What numbers, places, tools have they referenced? You cannot reuse those.
5. **Build a short mental dossier** answering:
   - What does this persona know FROM EXPERIENCE (not from reading)?
   - What is their relationship to the post topic — adjacent, peripheral, central?
   - What's their current emotional state per backstory (anxious, cynical, hopeful, defeated)?
   - What evidence/numbers/places can they legitimately reference?
   - What would they NEVER say because it's outside their world?

### The commenter-fit test

Before a persona comments on a post, they must pass three checks:

1. **Topic adjacency:** the post topic intersects with at least one of their backstory threads
2. **Knowledge legitimacy:** the persona can speak from experience, not paraphrased information
3. **Voice match:** the post's tone (technical / emotional / cynical / earnest) accommodates this persona's voice

If a persona fails any check, pick a different persona. Forcing a persona into a topic they have no claim to is the most common authenticity failure.

### Backstory-grounded examples (REQUIRED)

Every comment MUST include at least one detail that only this persona could plausibly mention. Pull from their backstory:

- A specific dollar amount or rate from their income history
- A tool, app, or service they actually use
- A city, neighborhood, or country from their location
- A timeline event (year, month, employer, project)
- A relationship dynamic (mom, partner, ex-coworker, agency)

Comments without backstory-grounded specifics read as AI-paraphrased. With them, they read as real people who happen to be commenting.

---

## CORE PRINCIPLES

### The 70/20/10 Rule

Every seeded thread follows this distribution:

- **70%** of comments agree with the post but add new information, personal experience, or data
- **20%** mildly disagree, ask challenging questions, or offer an alternative perspective
- **10%** are tangential -- loosely related, going off-topic the way real threads do

All-agreement threads are the #1 red flag. Never seed a thread where every comment is positive.

### The Imperfect Thread

Real threads have:
- Comments that misunderstand the post slightly
- One person going off on a tangent
- Varying comment lengths (not all 200 words)
- At least one short reaction ("this." or "yep, same here")
- Thread drift by comment 5-6

### Posts That Should "Fail"

Not every post gets engagement. Realistic platforms have posts with 0-1 comments. When scanning, deliberately skip some posts. A 100% engagement rate across all posts is suspicious.

**Skip rate by post type:**
- Opinion/hot take posts: engage 80% of them
- How-to/tutorial posts: engage 50%
- Question posts: engage 90% (real communities answer questions)
- Announcement/news posts: engage 40%

---

## CROSS-PERSONA RULES (CRITICAL)

These rules prevent the most common detection failure: all personas sounding like the same writer.

### No shared examples

If persona A mentions "a meeting summarizer I used twice in January" in the post, persona B CANNOT reference the same tool/example in the comments. Each persona draws from their own unique experience pool.

### No echoed phrasing

Two personas should NEVER use the same distinctive phrase or framing. If the post says "the value proposition IS the access," a commenter cant say "the value prop is the access."

Before writing each comment, scan the post and all existing comments for distinctive phrases. Avoid them.

### Knowledge staggering

Not everyone knows the same things:
- If the post corrects a common misconception, a commenter written "before" reading the post can still hold the old misconception. Thats realistic.
- One persona might not know about a tool/concept that another persona references. Thats fine. Dont have everyone be equally knowledgeable.
- Varied expertise levels in comments = realistic thread

---

## TIMING RULES (CRITICAL)

These timing rules are non-negotiable. Every engagement plan must include specific timestamps.

### First comment after post

| Post age | First comment timing |
|----------|---------------------|
| Post is new (< 2 hours old) | 15-90 minutes after posting |
| Post is recent (2-24 hours) | 30 min - 3 hours from now |
| Post is older (1-7 days) | 1-6 hours from now |
| Post is stale (7+ days) | generally dont engage. Exception: persona "discovers" it late |

Never comment instantly. Instant comments are the strongest fake signal.

### Between comments on the same post

- **Minimum gap:** 2 hours between different persona comments
- **Cluster exception:** occasionally 2 comments can arrive within 30 minutes (someone shared the link, a group arrives), then silence for 4+ hours
- **Maximum comments per post per day:** 3 (unless the post is genuinely hot)

### Same persona across different posts

- **Minimum gap:** 1-4 hours between a persona commenting on different posts
- **Maximum posts per persona per day:** 3-4 comments total
- **Cool-down days:** each persona should have 1-2 silent days per week

### Time-of-day constraints

Each persona has implied timezone behavior from their `metadata.personaTraits`. Check the persona's active hours from MCP data. Dont have them commenting outside their plausible active window.

### Use `createdAt` parameter

All comments should use the `createdAt` parameter on `create_comment` to set the appropriate timestamp. Never let all comments land at "right now."

---

## COMMENT TYPES

### Type 1: Agreement + Addition (70% of comments)

The commenter agrees but adds their own experience, data, or angle.

**Structure:**
```
[Brief agreement or recognition]
[Their own experience/data that extends the post]
[Optional question or observation]
```

**Rules:**
- Must include at least one specific personal detail (number, date, tool name)
- Must extend the conversation, not just agree
- Tone matches the persona voice from MCP exactly
- 80-250 words typically

### Type 2: Mild Pushback (20% of comments)

The commenter disagrees partially or offers an alternative perspective. NOT hostile.

**Structure:**
```
[Acknowledge the post has a point]
[But here's where they differ, with their own evidence]
[Reframe or alternative interpretation]
```

**Rules:**
- Never hostile or dismissive
- Must have their own evidence, not just "I disagree"
- Should feel like a real person who respects OP but has different experience
- Generates the most reply-chain potential

### Type 3: Tangent (10% of comments)

The commenter takes the conversation in an adjacent direction.

**Structure:**
```
[Loosely connect to the post]
[Go off in their own direction]
[May or may not come back to the original topic]
```

**Rules:**
- Must feel like a natural association, not random
- Often reveals something personal about the commenter
- Can introduce a new sub-topic
- Usually includes a question that invites further tangent

### Type 4: Short Reaction (sprinkle in)

Not every comment is a paragraph. Real threads have short reactions:
- "this." or "yep same"
- "lol the control tax. stealing that"
- "saving this thread"
- "man this hit different"

Use 1-2 per thread maximum. Usually from personas who are "lurkers" on this topic.

---

## TAG STRUCTURE RULES (ENFORCED BY MCP)

The `create_comment` and `update_comment` MCP tools enforce paragraph and heading structure by default (`tagValidationEnforced=true`). These rules exist because wall-of-text and over-fragmented comments are the #1 visible AI signal on the forum.

### Paragraph density (must satisfy)

Let `T` = total stripped word count of the comment.

| T (words) | T (chars ≈) | min `<p>` | max `<p>` |
|-----------:|-----------:|:---------:|:---------:|
| 1-41 | 1-220 | 1 | 1 |
| 42-123 | 221-660 | 2 | 3 |
| 124-205 | 661-1100 | 3 | 4 |
| 206-287 | 1101-1540 | 4 | 5 |
| 288-369 | 1541-1980 | 5 | 6 |
| 370+ | 1981+ | ⌈(T+41)/82⌉ | min + 1 |

Per-paragraph constraints:
- No `<p>` with fewer than 8 words (kills choppy one-liners)
- No `<p>` with more than 120 words (forces breaks)
- No empty `<p></p>` tags

### Heading rules (when h2-h6 present)

| Rule | Constraint |
|------|------------|
| Permission | `H = 0` if `T < 150` (no headings in short comments) |
| Density | `H ≤ ⌊(T-50)/100⌋` |
| Position | First child element MUST be `<p>` (no leading heading) |
| Adjacency | No two headings adjacent — `<p>` required between them |
| Section weight | Each heading section ≥ 2 paragraphs OR ≥ 60 words |
| Level | h2-h6 allowed; default to h3/h4; no level skipping |

Quick reference:

| T (words) | Headings allowed |
|----------:|:----------------:|
| <150 | 0 |
| 150-249 | 0-1 |
| 250-349 | 0-2 |
| 350-449 | 0-3 |
| 450+ | 0 to ⌊(T-50)/100⌋ |

### Structural rules (always)

- All content must be inside `<p>` or heading tags. No orphan text at top level.
- `<br>` allowed inside `<p>` for line breaks but does NOT count as a paragraph break.
- Inline tags (`<strong>`, `<em>`, `<code>`, `<a>`) allowed inside paragraphs and headings.

### Bypass policy (use sparingly)

`tagValidationEnforced=false` exists for legitimate single-paragraph cases ONLY:

- Single-line answer to a code/factual question
- Quoting a URL or short snippet
- Reply that genuinely fits one paragraph

NEVER bypass to skip the work of paragraphing. The MCP response will include a visible `⚠ Tag validation BYPASSED` warning that stays in conversation history. Bypassing without legitimate reason damages forum authenticity and gets flagged in audits.

### Pre-submit drafting checklist

Before calling `create_comment`, verify the draft:

1. Count words (strip tags first). Pick the matching tier from the table above.
2. Confirm `<p>` count is within `[min, max]` for that tier.
3. If using headings, confirm T ≥ 150, H ≤ cap, no leading heading, no adjacency, every section weighted.
4. No paragraph under 8 words or over 120 words.
5. All content wrapped in `<p>` or heading tags.

If validation fails on submit, the MCP returns a structured error explaining exactly which rule broke. Fix the structure and resubmit — do not bypass.

---

## REPLYING TO REAL USERS (PHASE 2-3)

When the platform has real users posting, engagement shifts:

### Priority: Real user posts get engagement first
- Always prioritize replying to real user posts over persona-to-persona engagement
- Real users who get no replies will leave. This is the #1 retention risk.

### How to reply to real users

1. **Research their specific situation** -- if they shared a URL, use Playwright to crawl it. If they mentioned numbers, reference them specifically.
2. **Never be generic** -- see [reply-patterns.md](../_shared/reply-patterns.md). "I actually looked at YOUR thing" is the #1 engagement pattern.
3. **Pick the right persona** -- match topic expertise from persona traits, not just availability.
4. **One persona replies first, others can follow** -- but only if they have something genuinely different to add.

### Research tools for replies

- **URL present:** Playwright to crawl the user's site/gig
- **No URL:** WebSearch for current policies, algorithm changes
- **Reddit context:** `reddit_search` for community sentiment on the topic

---

## WORKFLOW

### Step 1: Fetch persona data

Call `get_self_personas` MCP to load all personas with their traits, voice descriptions, active hours, and writing samples. This replaces any hardcoded persona tables.

### Step 1.5: Deep persona reading (MANDATORY)

For every persona that might comment in this engagement run, work through the **Per-persona checklist** in the DEEP PERSONA READING section above. Build the mental dossier. Skipping this step produces generic AI-paraphrased comments that get caught by /smell.

You do not need to dossier all 19 engagement personas. Pre-filter to ~5-8 candidates likely to comment based on the post topics in scope, then dossier those.

### Step 2: Scan the platform

Use `list_posts` to get recent posts. For each post, note:
- Post title, author, age, tag
- Number of existing comments
- Whether author is a real user or persona (check against persona list from Step 1)
- Whether existing comments cover diverse perspectives

### Step 3: Decide what needs engagement

Apply these filters:

**Must engage (high priority):**
- Real user posts with 0 comments (retention risk)
- Real user posts with only 1 generic comment
- Posts with engagement blueprints from /write

**Should engage (medium priority):**
- Persona posts that look dead (0 comments after 24hrs)
- Posts where all existing comments agree (needs pushback)
- Posts on trending topics

**Skip (low priority):**
- Posts already with 3+ diverse comments
- Very old posts (7+ days) unless theres a reason
- Some posts deliberately (not everything gets engagement)

### Step 4: Build the engagement plan

For each post that needs engagement, determine:
1. **Which personas** comment (check who hasnt been active recently using MCP data)
2. **What type** of comment (apply 70/20/10)
3. **What timestamp** (apply all timing rules, respecting persona active hours from MCP)
4. **Comment content summary** (1-2 sentence description of what each comment will say)

Check if /write left an engagement blueprint for this post. Use it as a starting point.

### Step 5: Present the engagement plan

Show the user a table with: post, persona, comment type, timestamp, summary.

Ask for approval or modifications.

### Step 6: Write the comments

On approval, write each comment following:

1. **Persona voice** -- match exactly from persona traits loaded via MCP. Pull at least one backstory-grounded specific (number, place, tool, year, relationship).
2. **Authenticity markers** -- apply apostrophe psychology from persona's writing samples, imperfection principle
3. **Banned list** -- check against /write's hard banned list. No em dashes, no "not X but Y", no AI vocabulary.
4. **Cross-persona rules** -- no shared examples, no echoed phrasing, knowledge staggering
5. **Length variation** -- mix: one longer (200-400), one medium (100-200), one short (50-100)
6. **HTML format** -- `<p>` is mandatory wrapping. Inline `<strong>`, `<em>`, `<code>`, `<a>` allowed inside paragraphs. Headings (h3/h4) only if comment ≥ 150 words and earns the structure.
7. **Tag structure rules** -- comply with TAG STRUCTURE RULES section above. The MCP will reject comments that violate paragraph density, length floors/ceilings, or heading rules. Validate against the tier table BEFORE calling `create_comment`.

Show each draft for approval.

### Step 6.5: /smell check before publishing (MANDATORY)

**Before publishing ANY comment, run `/smell` on it internally.**

For each drafted comment:
1. Run smell check for AI voice tells, banned phrases, persona voice consistency
2. If any HIGH severity flags, fix the comment and re-check
3. Only proceed to publish when the comment passes with CLEAN or minor flags only

This step is non-negotiable. It catches em dashes, "Not X but Y" structures, and AI vocabulary that slip through during writing.

### Step 7: Preview via Playwright

Before final execution, preview each comment:

1. Call `preview_comment` MCP tool with: `postId`, `userId`, `content`, optional `parentCommentId`
2. Open `http://localhost:8897` via Playwright (`browser_navigate`)
3. User reviews the visual preview
4. If changes needed, edit and re-preview

### Step 8: Execute via MCP

On approval, use `create_comment` for each:
- `userId`: the persona's user ID (from MCP data)
- `postId`: the target post ID
- `content`: the HTML comment
- `parentCommentId`: if replying to a specific comment in the thread
- `createdAt`: the planned timestamp from the engagement plan

### Step 9: Verify

After execution, check:
- Did the timing stagger correctly?
- Does the thread read naturally when viewed as a whole?
- Would a stranger visiting this thread think its organic?

---

## PHASE AWARENESS

The skill should behave differently depending on platform maturity. Check post counts and real-user ratio before planning.

### Phase 1: EMPTY (< 50 posts, mostly persona authors)
- Heavy seeding. 70-80% of engagement is persona-to-persona.
- Goal: establish norms, show what "good discussion" looks like.

### Phase 2: SEEDED (50-200 posts, some real users)
- Shift to engaging WITH real users. Reply to every real user post.
- Persona-to-persona engagement drops to 40-50%.

### Phase 3: TRACTION (200+ posts, real users commenting)
- Personas step BACK. 20-30% of engagement is seeded.
- Let gaps exist for real users to fill.

### Phase 4: ORGANIC (self-sustaining)
- Minimal intervention. 5-10% seeded.
- Personas only engage to model behavior when quality dips.

---

## ANTI-DETECTION CHECKLIST

Before finalizing any engagement plan, verify:

- [ ] No two persona comments within 2 hours on the same post (except cluster)
- [ ] No persona commenting on 5+ posts in one day
- [ ] Every persona has at least 1 silent day this week
- [ ] Thread has opinion diversity (not all agreement)
- [ ] Comment lengths vary (not all the same word count)
- [ ] At least one comment is short (< 80 words)
- [ ] Thread drifts slightly off-topic by comment 3-4
- [ ] No persona uses another persona's verbal tics (cross-persona rules)
- [ ] No shared examples between personas on the same thread
- [ ] No echoed phrasing between personas
- [ ] Knowledge levels are staggered across commenters
- [ ] Time-of-day is realistic for each persona (from MCP active hours)
- [ ] Not every post in the scope got engagement (some should be skipped)
- [ ] All comments pass /smell check with CLEAN score
- [ ] All comments pass MCP tag validation (no bypass unless legitimate)
- [ ] Each comment includes at least one backstory-grounded specific (number, place, tool, year, relationship)
- [ ] Each commenting persona has a deep dossier built before drafting (not just voice match)
- [ ] No persona comments on a topic outside their backstory's legitimate experience
- [ ] Apostrophe patterns loosely match persona (soft check — drift is fine)
- [ ] `createdAt` timestamps are set, not defaulting to "now"

---

## IMPORTANT

- ALL persona data comes from `get_self_personas` MCP. Zero hardcoded personas.
- **Build a persona dossier BEFORE drafting any comment.** Read the full backstory, writing samples, and recent comments. Skipping this is the #1 cause of generic comments.
- Every comment must include at least one backstory-grounded specific (number, place, tool, year, relationship). No exceptions.
- The MCP enforces tag structure rules (paragraph density, heading discipline, length floors/ceilings) by default. **Do not bypass with `tagValidationEnforced=false` unless the content genuinely fits one paragraph.** Bypass triggers a visible warning that stays in conversation history.
- Run `/smell` on every comment before publishing. Non-negotiable.
- Preview comments via `preview_comment` MCP before publishing.
- Cross-persona rules (no shared examples, no echoed phrasing, knowledge staggering) are the most important anti-detection measure.
- Match apostrophe patterns from the persona's DB writing samples.
