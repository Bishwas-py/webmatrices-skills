# Universal Content Blocks — Shared Reference

Hard rules that apply to **every** content-generation skill, every mode, every persona, every target platform. No exceptions for editorial style, long-form, admin replies, or "the persona's natural voice." These are the AI-tells that ship the whole output to the spam folder of human attention.

Referenced by: `write-post`, `write-reply`, `write`, `smell`, `reduce-smell`, `simulate-engagement`.

---

## Why these are universal

Sample-based voice (sentence shape, casing, slang, punctuation rhythm) is **per-persona** — it follows whatever the persona's `metadata.personaTraits.writingSamples` shows. Universal blocks are the small set of patterns that always smell like AI, regardless of which persona is writing or what platform they're posting to. They override sample mimicry — if a persona's samples contain a banned pattern, we still strip it from output (and flag the sample for cleanup).

---

## The blocks

### 1. No "Honestly," as a sentence opener

`Honestly, the real issue is...` reads as AI scaffolding. Mid-sentence "honestly" is fine: `i think honestly the problem is...`. Only the opener is banned.

| BAD | GOOD |
|-----|------|
| Honestly, the pricing is the problem. | the pricing is the problem honestly. |
| Honestly? I tried it and hated it. | tried it and hated it. |

### 2. No hyphen `-` as clause punctuation

The plain hyphen used to separate clauses (with or without surrounding spaces) is AI rhythm. It's a soft em-dash substitute. Banned.

| BAD | GOOD |
|-----|------|
| the real problem - and nobody talks about this - is pricing | the real problem is pricing. nobody talks about it. |
| i tried it -worked fine | i tried it. worked fine. |
| stack overflow is dead - LLMs killed it | stack overflow is dead. LLMs killed it. |

**Allowed**: hyphens INSIDE compound words. `well-known`, `long-term`, `self-promotion`, `vibe-coding`, `open-source` are fine. The rule is: a hyphen must join two words into ONE term. If it's joining clauses, it's banned.

### 3. No em-dashes (—)

The single most reliable AI signature in 2025-2026. Every em-dash dies, no exceptions.

| BAD | GOOD |
|-----|------|
| i looked at the numbers — they don't add up | i looked at the numbers. they don't add up. |
| this works — most of the time | this works most of the time. |

### 4. No polished essay-style prose

Real comments and posts are messier than essays. Real writing has:
- fragments
- lowercase starts
- dropped articles ("checked dashboard, nothing there")
- run-on thoughts
- casual swears where the persona's samples have them

Banned formal connectors (anywhere — opener, mid-paragraph, closer):

- `Furthermore`
- `Additionally`
- `Moreover`
- `In conclusion`
- `In summary`
- `That said,` (as a paragraph opener)
- `It's worth noting`
- `Interestingly`
- `Notably`
- `Specifically,` (as an opener)

Banned essay-mode patterns:

- "Not X, but Y" parallel structures
- "Not X. Not Y. Z." triple-repetition rhythm
- "Let that sink in" / "Read that again"
- "Let me get this straight"
- "I'll say it louder" / "Say it louder for the people in the back"
- "At the end of the day" (cliché filler)
- "Needless to say" (AI padding)
- "It goes without saying"
- "In my humble opinion"

### 5. No AI vocabulary

These words land like a flare. Replace with the persona's natural word choices from their samples.

| BAD | GOOD |
|-----|------|
| comprehensive | full / complete / covers everything |
| robust | solid / works / holds up |
| streamline | speed up / cut steps / simplify |
| delve | look at / dig into |
| leverage | use |
| crucial | matters / important |
| game-changer | changed things / made a difference |
| paradigm shift | shift / change |

### 6. No performed emotion

Don't announce emotion. Show it through specific mundane detail.

| BAD | GOOD |
|-----|------|
| This is the part that made me close my laptop and go for a walk. | i read this on my phone at 1am and couldn't sleep. |
| What absolutely blew my mind was... | the thing i didn't expect: |

### 7. No engagement bait

| BAD | GOOD |
|-----|------|
| What do you think? | (delete; or ask a real specific question) |
| Let me know in the comments! | (delete) |
| Drop a 🔥 if you agree | (delete) |

---

## What is NOT a universal block

The following are **per-persona** patterns driven by `writingSamples`. They are NOT universal rules — they shift with each persona.

- **Casing**. If samples are lowercase, output lowercase. If samples are standard caps, use standard caps. No universal rule.
- **Apostrophes**. Whatever the samples show. `don't` vs `dont` is per-persona, soft preference, drift tolerated.
- **Length**. Reply length is anchored to sample average per persona. Post length is platform/sub-driven. No universal length rule.
- **Sentence shape, slang, punctuation rhythm**. All sample-driven.

---

## How skills use this file

- **`write-post`, `write-reply`** — reference this file in their workflow as the hard-banned list. Apply checks before presenting any draft.
- **`smell`** — every block above is a HIGH severity detector. Flag with paragraph number + exact quote.
- **`reduce-smell`** — every block has a 1:1 fix recipe (see Fix Recipes below).

---

## Fix recipes (for reduce-smell)

| Block | Fix |
|-------|-----|
| "Honestly," opener | Delete "Honestly, " entirely. If the sentence collapses, rewrite from scratch with the same meaning in the persona's sample voice. |
| Clause-hyphen | Split into two sentences at the hyphen. Or replace with a comma if the clauses are tightly coupled. |
| Em-dash | Same as clause-hyphen. Split into two sentences. |
| Formal connector | Delete the connector and either let the section shift be abrupt (more human), or replace with the persona's natural connector pattern from samples. |
| "Not X, but Y" | Rewrite as two statements: "X is wrong. Y is what actually happens." |
| AI vocabulary | Look up the persona's `writingSamples` for how they would express the same idea. If no analog, replace with the plain-English version from the table above. |
| Performed emotion | Replace announced emotion with a specific mundane detail. The detail should come from the persona's backstory if possible. |
| Engagement bait closer | Delete the closing question entirely, or replace with a specific question the persona genuinely wants answered ("anyone else seeing this on small sites or is it just my niche?"). |

---

## Exceptions

There are no exceptions to the universal blocks. Previous versions of the `write` skill allowed em-dashes for the admin persona's help replies. **That exception is revoked.** Admin replies follow the same blocks. If the admin persona's natural voice "uses em-dashes," the samples need updating, not the rules.
