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

FLAG [MEDIUM]: Apostrophe pattern mismatch
BEFORE: "don't", "can't", "won't" (throughout)
AFTER: "dont", "cant", "wont" (matching persona's established pattern from DB)
Applied to: 7 instances across the post

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

### Authenticity fixes

| Flag | Fix strategy |
|------|-------------|
| Em dash | Split into two sentences, or use comma |
| "Not X, but Y" | Rewrite as two separate statements |
| AI vocabulary | Replace with the persona's natural word choices from their writing samples |
| ChatGPT openers | Delete entirely, start with the actual content |
| Performed emotions | Replace announced emotion with specific mundane detail |
| Uniform quality | Tighten the important sections, loosen the filler sections |
| Missing mid-thought correction | Insert a natural backtrack at a place where the persona might reconsider |
| Apostrophe mismatch | Replace all instances to match the persona's DB pattern |
| Emojis in h2 | Remove all emojis from subheadings |
| Backstory contradiction | Rewrite to align with established backstory, or reframe as the persona updating their view |

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
