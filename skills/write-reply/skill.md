---
name: write-reply
description: Write short, casual, sample-shaped replies as a Webmatrices persona — comments on posts, replies to comments, HN replies, Reddit replies. Anchors length and voice to the persona's writing samples. Always short, never essay-prose. Publishes via MCP after approval. Use when asked to reply, comment, or respond as a persona.
disable-model-invocation: true
argument-hint: [persona] [postId/commentId/"quoted text"]
---

# Write Reply

Write a short reaction as a persona. Replies are messier and shorter than posts. The persona's writing samples are the only voice anchor. No engagement formulas, no subreddit conventions, no essay structure. Universal content blocks always apply.

For the universal block list, see [_shared/universal-blocks.md](../_shared/universal-blocks.md).
For the publishing workflow, see [_shared/publishing.md] *(if missing, fall back to the inline publish step below)*.

---

## Subcommands

| Command | What it does |
|---------|-------------|
| `/write-reply [persona] [postId]` | Reply to a Webmatrices post as that persona |
| `/write-reply [persona] comment [commentId]` | Reply to a specific comment in a thread |
| `/write-reply [persona] hn "[quoted text]"` | Hacker News reply to quoted text |
| `/write-reply [persona] reddit "[post body]"` | Reddit comment to a post body |
| `/write-reply preview` | Preview the current draft via `preview_comment` MCP + open in Chrome |
| `/write-reply publish` | Publish via `create_comment` MCP |

If no persona is provided, auto-match using audience-matcher logic AFTER fetching personas from MCP.
If no args at all, ask what to reply to.

---

## CORE RULE

> **Samples are the voice. Backstory is the perspective. Length is anchored to the samples. Nothing else applies.**

The persona's `metadata.personaTraits.writingSamples` drives:

- sentence shape
- casing (lowercase, sentence case, mixed — whatever samples show)
- punctuation rhythm
- slang and casual swears (only if samples use them)
- apostrophe usage (soft — drift is fine)
- length

The persona's `backstory`, `topics`, `opinionStrength`, `signaturePhrases` drive:

- whether the persona would react at all
- what stance they would take
- whether they have lived experience with the topic
- which signature phrase (if any) would naturally surface

**Nothing else** — no engagement formulas, no hook engineering, no gap principle, no platform conventions, no emotional pacing. Replies are reactions, not designed content.

---

## WORKFLOW

### Step 1: Fetch persona and target

1. Call `get_self_personas` to fetch all personas with their traits.
2. If persona name provided, match it.
3. If no persona, auto-match: which persona would actually react to this thread based on their `topics` and `backstory`? If no persona naturally fits, **don't force one**. Tell the user.
4. Fetch the target:
   - postId → `get_post` MCP for post body + `get_comments` MCP for thread context
   - commentId → `get_comments` MCP for parent chain
   - HN / Reddit raw text → use the provided text directly

### Step 2: Measure sample length

Compute the average word count of the persona's reply-style samples:

- Prefer `metadata.personaTraits.writingSamples.replies` if present (a flat array of 3 short example replies the persona has made)
- If `.replies` is missing, fall back to any short-form samples in `writingSamples` (e.g., individual sentences from `signatureRhythm`)
- If only post-shaped samples exist (openers, closers), warn the user: this persona has no reply samples — output will be best-effort.

Set the target length: **sample average × 1.2, default 1-3 sentences.** Hard cap at sample average × 1.5.

### Step 3: Read the context

Read the live post body and the parent comment chain. The reply is a reaction to **this specific thread**, not a designed piece of content. The persona is seeing this on their feed and deciding whether to say something.

Do NOT inject:

- subreddit conventions
- engagement hooks
- the Gap Principle
- a "debatable claim" plant
- closing questions designed for engagement

The reply is whatever the persona would naturally say after reading the thread. If it's a one-liner reaction, it's a one-liner.

### Step 4: Decide whether to react

Use the persona's `opinionStrength` and `topics`:

- HARD opinions + topic match → they react, with stance
- SOFT opinions + topic match → they react, mild
- No topic overlap + no backstory hook → **skip**. Don't manufacture engagement.

If the persona has no organic reason to comment, the right output is: "this persona wouldn't actually reply here. Pick a different persona or skip."

### Step 5: Draft the reply

Match the samples directly:

- Sentence shape (fragments, run-ons, mid-clause stops)
- Casing
- Apostrophe pattern (soft)
- Slang and swears (only from samples)
- Length (within the cap from Step 2)

Backstory surfaces ONLY if naturally relevant to the thread. Don't force "as someone who has X years of experience in Y" — that's the AI tell of credentialing.

### Step 6: Apply universal blocks

Run the draft through every block in [_shared/universal-blocks.md](../_shared/universal-blocks.md):

- [ ] No "Honestly," opener (mid-sentence fine)
- [ ] No hyphen `-` as clause punctuation (compound words like `well-known` are fine)
- [ ] No em-dash (—)
- [ ] No formal connectors (`Furthermore`, `Additionally`, `Moreover`, `In conclusion`, `That said,` opener, `Specifically,` opener, `Interestingly`, `Notably`, `It's worth noting`)
- [ ] No "Not X, but Y" parallel
- [ ] No "Not X. Not Y. Z." triple-repetition
- [ ] No AI vocabulary (`comprehensive`, `robust`, `streamline`, `delve`, `leverage`, `crucial`, `game-changer`)
- [ ] No performed emotion ("This made me close my laptop...")
- [ ] No engagement bait closer ("What do you think?", "Let me know!", "Drop a 🔥")
- [ ] No "Fair point" / "Great question!" / "Absolutely!" openers (ChatGPT residue)

### Step 7: Length check

- Count words in draft.
- Is it within sample average × 1.2? If not, cut.
- Default target: 1-3 sentences.
- If the draft is 50+ words and the persona's samples average 15 words, you've overwritten. Cut to one or two sentences.

### Step 8: Present to user

Show:

```
PERSONA: [username] ([why this persona reacts])
TARGET:  [post title / comment snippet]
LENGTH:  [N words] (sample avg: [M words], cap: [M*1.2])
SAMPLES USED: [first 80 chars of the 1-3 samples that anchored voice]

DRAFT:
[the reply]

BLOCK CHECK: ✓ all clear (or list any failures)
```

Ask: approve, edit, or skip.

### Step 9: Preview (optional, on `/write-reply preview`)

For Webmatrices replies:

1. Call `preview_comment` MCP with `postId`, `userId`, `content`, optional `parentCommentId`
2. Open `http://localhost:8897` via Playwright (`browser_navigate`)

For HN / Reddit / raw text: skip preview, just show the draft.

### Step 10: Publish (on `/write-reply publish`)

For Webmatrices:

1. Call `create_comment` MCP with `userId`, `postId`, `content`, optional `parentCommentId`, `createdAt`
2. Display result (comment ID, post URL)

For HN / Reddit / raw text: copy-paste output. This skill does not publish off-platform.

---

## PROJECT BLOCK (promote intent)

When a PROJECT block is provided in the invocation:

```
PROJECT:
  name: [project name]
  pitch: [one-line pitch]
  url: [url]
  style: subtle | story | direct | question
```

Mention the project **once, naturally**, in the voice of the samples. Rules:

- Do **not** paste the pitch verbatim.
- Do **not** gate on relevance. If the user provided the PROJECT block, they want it mentioned.
- The mention must read as something the persona would actually say, not a plug.
- Style hints how to weave it in:
  - `subtle` — slip the name in as a passing reference, no link unless context demands it
  - `story` — wrap the mention in a one-line personal observation ("i built [name] last month for this exact problem")
  - `direct` — name + what it does, one short sentence
  - `question` — frame as the persona asking the thread whether they've tried it ("anyone using [name] for this?")

The mention counts against the length budget. If the sample cap is 30 words and the project mention is 15 of those, the rest of the reply is 15 words.

---

## REPLY-MODE SPECIFICS

### Webmatrices comment on a post

- Default to 1-2 sentences unless the post asks a direct question
- HTML format: `<p>`, `<strong>`, `<a>` only. No headers. No lists.
- Must include at least one persona-specific detail if the reply runs >2 sentences (a number, a project name, a specific past experience). For 1-sentence reactions, this is optional.

### Reply to a comment in a thread

- Read the full parent chain.
- The reply is to the comment directly above, not to the original post.
- Match the depth of the parent comment's energy. If the parent is one short sentence, your reply is one short sentence.

### Hacker News reply

HN readers are the most AI-aware audience. Apply the universal blocks **plus**:

- One paragraph only. No structure, no headers, no lists.
- Open with an anecdote or specific experience. NEVER with agreement ("Fair point", "Good point", "I agree", "Absolutely").
- Reference specific personal experience, not abstract agreement.
- Tone: conversational, slightly tired, specific. Like you're talking to a smart friend at a bar.
- Max 4-5 sentences.
- No sign-off, no closing question, no "happy to discuss further."

### Reddit comment

- Sample-shaped voice. No subreddit-specific conventions injected.
- The thread itself is the only context. Don't try to match "the sub's vibe" — match the samples.
- Length anchored to samples, default 1-3 sentences. Reddit comments routinely run 1 sentence; don't pad.

---

## IMPORTANT

- **Samples are the voice.** Backstory drives stance, never voice. If the output sounds like an essay, the backstory drove voice — go back and re-anchor to samples.
- **Short by default.** If you're writing more than 3 sentences, you're probably writing a mini-post. Cut.
- **No engagement design.** Replies are reactions, not designed content. Skip the Gap Principle, hooks, engagement blueprints — those belong in `/write-post`.
- **Universal blocks override sample mimicry.** Even if a persona's samples contain an em-dash, output drops it. Flag the sample for cleanup.
- **One generation per request.** Generate, run the block check, present. Do not silently re-draft. If the draft fails the block check, fix it once and present. If it still fails, tell the user.
- **Don't force a persona.** If no persona naturally fits the topic, say so. Skipping is better than approximating.
