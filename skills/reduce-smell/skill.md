---
name: reduce-smell
description: Fix authenticity and quality problems found by /smell. Runs /smell internally, generates specific fixes with diffs, and previews the result. NEVER publishes. Use when asked to fix AI tells, reduce smell, clean up content, or make content more authentic.
disable-model-invocation: true
argument-hint: [postId/slug/commentId/"raw text"]
---

# Reduce Smell

Fix authenticity, factual, and quality problems detected by `/smell`. This skill runs `/smell` internally, generates targeted fixes for every flag, and previews the result. It NEVER publishes or updates content directly.

**STRICT RULE: This skill NEVER publishes. It only suggests fixes and opens a preview. The user must run `/write publish` or manually approve any changes.**

---

## INPUT

Same as `/smell`:

| Input | What it does |
|-------|-------------|
| Post ID | Fetches post via `get_post` MCP, runs smell, generates fixes |
| Post slug | Fetches via `get_post_by_slug` MCP |
| Comment ID | Fetches comment via `get_comments` MCP |
| Quoted text ("...") | Runs smell on raw text, generates fixes inline |

---

## WORKFLOW

### Step 1: Run /smell

Invoke the `/smell` skill internally on the provided input. Collect all flags across all three categories (AUTHENTICITY, FACTUAL, QUALITY).

### Step 2: Fetch persona data

Call `get_self_personas` MCP to load the author's persona traits. This is needed to:
- Match apostrophe patterns during fixes
- Maintain voice consistency
- Keep backstory alignment

### Step 3: Generate fixes for each flag

For every flag in the smell report, generate a specific fix shown as a diff:

```
FLAG [HIGH]: Em dash found in paragraph 3
BEFORE: "the real problem -- and nobody talks about this -- is the pricing"
AFTER:  "the real problem is the pricing and nobody talks about it"

FLAG [HIGH]: Banned phrase "Not X, but Y" in paragraph 5
BEFORE: "its not about the backlinks, but about the authority signals"
AFTER:  "backlinks are a distraction. the authority signals are what actually move the needle"

FLAG [HIGH]: Uniform section quality
FIX: Tighten section 2 (the core argument -- this is what the persona cares about).
     Loosen section 4 (supporting context -- let this feel more rushed, shorter paragraphs, less polished).

FLAG [HIGH]: Paragraph 4 and 8 repeat the same point
FIX: Delete paragraph 8 entirely. Paragraph 4 already makes this point.

FLAG [MEDIUM]: No mid-thought correction anywhere
FIX: Add to paragraph 6: "actually wait, i looked at this again and the numbers are slightly different than what i said earlier"

FLAG [MEDIUM]: Missing source links
FIX: Add link to official docs in paragraph 2 where Vercel encryption is mentioned.
     Add link to the research paper in paragraph 5.
```

### Step 4: Apply fixes and generate revised content

Apply all fixes to produce a complete revised version. Show the full revised content.

### Step 5: Re-run /smell on the fixed version

Run `/smell` again on the revised content to verify all flags are resolved. If new flags appeared, fix those too.

### Step 6: Preview

Open a preview of the fixed content:

**For posts:**
1. Call `preview_post` MCP with the revised: `title`, `body`, `authorId`, `tagSlugs`
2. Open `http://localhost:8897` via Playwright (`browser_navigate`)

**For comments:**
1. Call `preview_comment` MCP with the revised: `postId`, `userId`, `content`, optional `parentCommentId`
2. Open `http://localhost:8897` via Playwright (`browser_navigate`)

**For raw text:**
Skip preview. Just show the revised text.

### Step 7: Present to user

Show:
- Original smell score vs. revised smell score
- All diffs applied
- The complete revised content
- Preview URL (if applicable)

Then tell the user:
```
Preview is ready at http://localhost:8897
Review the changes, then:
- /write publish to publish the revised version
- Or request further edits
```

**NEVER auto-publish. NEVER call create_post or create_comment. NEVER call update_post or update_comment.**

---

## FIX STRATEGIES BY FLAG TYPE

### Surface authenticity fixes

| Flag | Fix strategy |
|------|-------------|
| Em dash | Split into two sentences, or use comma |
| "Not X, but Y" | Rewrite as two separate statements |
| AI vocabulary | Replace with the persona's natural word choices from their writing samples |
| ChatGPT openers | Delete entirely, start with the actual content |
| Performed emotions | Replace announced emotion with specific mundane detail |
| Missing mid-thought correction | Insert a natural backtrack at a place where the persona might reconsider |
| Apostrophe mismatch | Informational only — do not fix. Natural drift is expected and tolerated. |
| Emojis in h2 | Remove all emojis from subheadings |
| Backstory contradiction | Rewrite to align with established backstory, or reframe as the persona updating their view |

### Deep authenticity fixes (THE THIN LINE)

These are the structural fixes that separate "approximating" a persona from "inhabiting" them.

| Flag | Fix strategy |
|------|-------------|
| POLITE STRANGER | Rewrite the flagged sections using specific details from the persona's `backstory` in DB. Replace generic observations with details only THIS persona would know. Not "I worked on a project" but the specific project from their backstory (fetch from DB). Load their writing samples and match the rhythm. |
| EARNED OPINIONS | Trace the opinion back to the persona's backstory. If the persona says "vibe coding is dangerous," add the specific experience from their DB backstory that EARNED them that view. If no backstory supports the opinion, either add backstory context via `update_persona` MCP or soften the claim to speculation. |
| UNIFORM QUALITY / INVESTMENT MAP | Identify which section the persona cares most about (from their topics in DB). Tighten THAT section — shorter sentences, stronger verbs, more precise language. Then deliberately LOOSEN 1-2 other sections — longer sentences, more qualifiers, trailing thoughts. The quality variation should map to emotional investment. |
| THINKING PROCESS MISSING | Insert a visible reasoning journey. Not "The answer is X." but "I initially thought X. Then I looked at Y. Actually, Z might be closer to the truth." Place this at the natural inflection point of the argument — where the persona would genuinely reconsider. |
| TEMPLATE / GENERIC CONTENT | The hardest fix. If the content could work for any persona with minor voice changes, the problem is the CONTENT not the VOICE. Options: (a) Add a backstory-specific anecdote that only this persona could tell, (b) Reframe the argument through this persona's unique professional lens, (c) Add a specific number or detail from their backstory that grounds the abstract argument. |
| UNSAID / OVEREXPLAINING | Delete explanations the audience already knows. If writing for Webmatrices devs, cut "AdSense is Google's ad program." If writing for freelancers, cut "MRR means monthly recurring revenue." Trust the community's shared context. |
| TOO GOOD / TOO POLISHED | Introduce deliberate imperfection that matches the persona's style. For a confessional persona: shorter paragraphs, more fragments, one trailing "..." thought. For a data-backed persona: one self-correction mid-thought. For an uncertain persona: one unanswered question. Match the specific imperfection to the persona's voice from their DB traits. |

### Persona trait fixes

When `/smell` flags that the persona's TRAITS THEMSELVES are smelly, use the `update_persona` MCP tool to fix the source.

#### Internal backstory contradictions

When two backstory fields contradict each other, decide which one to keep based on what the persona has already published. Then fix the other.

**Strategy: Bridge with timeline, dont delete.**

WRONG approach:
```
Contradiction: location says "London" but timeline says "born in USA, never left"
Fix: delete location → set to "USA"
```

RIGHT approach:
```
Contradiction: location says "London" but timeline says "born in USA, never left"
Fix: add migration event to timeline → "born in USA, moved to London in 2019 for work"
     Now both facts coexist. The backstory gained a life event instead of losing one.
```

Always prefer ADDING a bridging event over DELETING a contradicting fact. Real people have complex histories. The fix should make the backstory MORE lived-in, not less.

```
update_persona MCP:
  userId: [persona ID]
  personaTraits: '{"backstory":{"timeline":["...existing events...", "2019: moved to London for a contract that became permanent"]}}'
```

#### Age vs experience math

If age and experience years dont add up (e.g. "39 years old" + "15 years experience since college" = started at 24, graduated at 24, plausible). But "29 years old" + "15 years experience" = started at 14, suspicious.

Fix: adjust whichever number the persona has referenced LESS in published posts. If theyve mentioned their age in 3 posts but experience years in 1 post, adjust the experience years.

#### Income vs struggles mismatch

If income is high but the persona claims financial struggles, add CONTEXT that explains it: student loans, supporting family, expensive city, divorce, medical bills. Dont just lower the income — real people can earn well and still struggle.

#### Backstory too clean (character sheet / convenience / missing mundane)

When `/smell` flags the backstory as too manufactured:

1. Add one IRRELEVANT detail that has nothing to do with their content topics (a hobby, a past job in a different field, a city they lived in briefly)
2. Add one UNRESOLVED thread (something they started and never finished, a question they still dont have an answer to)
3. Make one number less round ("14 years" → "started in 2012, so whatever that is now")
4. Add one growth moment ("I used to think X but changed my mind after Y")

```
update_persona MCP:
  userId: [persona ID]
  personaTraits: '{"backstory":{"tools":["...existing...", "used to play guitar, sold it during a move"], "struggles":["...existing...", "still havent figured out health insurance properly"]}}'
```

After ANY trait update, re-run `/smell` on the persona's recent posts to check for new inconsistencies between the updated backstory and published content.

### Factual fixes

| Flag | Fix strategy |
|------|-------------|
| Unverified claim | Run `/fact-check` on the specific claim. If wrong, rewrite. If unverifiable, reframe as persona's observation. |
| Stale pricing | Look up current pricing, update the number |
| Wrong technical detail | Reframe as persona's initial misunderstanding + correction (more interesting than getting it right silently) |

### Quality fixes

| Flag | Fix strategy |
|------|-------------|
| Repeated information | Delete the weaker instance, keep the stronger one |
| Filler paragraph | Delete entirely |
| Monotonous pacing | Vary paragraph lengths. Mix short punchy paragraphs with longer flowing ones |
| Redundant transitions | Delete the transition, let the section shift be abrupt (more human) |
| Same sentence structure | Rewrite alternating sentences to break the pattern |
| Overexplaining | Cut to the conclusion. Trust the reader. |
| Dead weight section | Delete or compress to 1-2 sentences |
| Too many links | Remove the weakest links, keep only primary sources |

---

## IMPORTANT

- This skill is a FIXER, not a PUBLISHER
- Always show diffs so the user can see exactly what changed
- Always re-run /smell on the fixed version to confirm improvement
- Always open a preview so the user can visually verify
- The user decides whether to publish, not this skill
