---
name: smell
description: Universal authenticity scanner for Webmatrices content. Scans posts, comments, user profiles, or raw text for AI tells, factual issues, quality problems, and cross-persona bleeding. Use when asked to audit content, check authenticity, detect AI writing, review engagement quality, or verify organic appearance.
disable-model-invocation: true
argument-hint: [postId/slug/commentId/username/"raw text"]
---

# Smell

Universal authenticity scanner. Detects AI tells, factual issues, quality problems, and cross-persona bleeding in any Webmatrices content. This skill absorbs the old detect-fake-engagement functionality and adds deeper content analysis.

No subcommands. Auto-detects what you're scanning based on the input.

For the canonical universal-block list (em-dash, "Honestly," opener, clause-hyphen, formal connectors, AI vocab, etc.), see [universal-blocks.md](../_shared/universal-blocks.md). Every block in that file is a HIGH severity detector here.

For engagement psychology research, see [engagement-psychology.md](../_shared/engagement-psychology.md).
For reply quality patterns, see [reply-patterns.md](../_shared/reply-patterns.md).

---

## CORE FRAMING (v2.1)

Voice is per-persona, anchored to `writingSamples`. Detectors compare output against the persona's samples for sentence shape, casing, punctuation rhythm, and length. Drift from samples is a smell.

Stance is per-persona, anchored to `backstory`. Detectors check whether opinions trace back to lived experience. Detached opinions (no backstory hook) are a smell.

**Do not confuse the two.** A persona that sounds wrong (sentence shape mismatch) has a *voice* smell. A persona that takes a stance they couldn't have arrived at (no backstory) has a *stance* smell. They have different fixes.

---

## INPUT DETECTION

| Input | What it does |
|-------|-------------|
| Post ID (UUID-like string) | Fetches post + comments via `get_post` MCP, scans everything |
| Post slug (contains hyphens, not UUID) | Fetches via `get_post_by_slug` MCP |
| Comment ID | Fetches comment thread via `get_comments` MCP |
| Username (starts with @) | Fetches user's posts via `list_posts`, scans for voice consistency |
| Quoted text ("...") | Scans the raw text directly without fetching |
| "recent" | Scans last 20 posts via `list_posts` |
| No args | Same as "recent" |

After fetching content, also call `get_self_personas` MCP to load all persona traits (voice, apostrophe patterns, backstory, opinion strength). This is needed to check voice consistency and cross-persona bleeding.

---

## FOUR SMELL CATEGORIES

Categories 1-3 are pattern-matching: scan for specific tells, known phrases, structural anti-patterns. Category 4 is experiential: read line by line as a human reader and flag every place attention drops. Run all four for post-mode scans.

### Category 1: AUTHENTICITY

Detects patterns that make content feel AI-generated or fake.

#### AI Voice Tells

| Signal | Severity | Example |
|--------|----------|---------|
| Em dash (—) in content | HIGH | "the real problem — and nobody talks about this — is..." |
| Hyphen `-` as clause punctuation | HIGH | "the problem - and nobody talks about this - is..." (compound words like `well-known` are exempt) |
| "Honestly," as a sentence opener | HIGH | "Honestly, the pricing is the issue." (mid-sentence "honestly" is fine) |
| "Not X, but Y" structures | HIGH | "its not about traffic, but about intent" |
| Bullet points in conversational comments | HIGH | Comments formatted as advice lists |
| Too helpful / too complete | MEDIUM | Comment covers every angle, leaves nothing to add |
| Too balanced ("on one hand... on the other") | MEDIUM | Hedging instead of taking a position |
| Perfect grammar throughout | MEDIUM | No contractions skipped, no casual errors |
| AI vocabulary | HIGH | "moreover", "crucial", "delve", "leverage", "comprehensive", "robust", "streamline" |
| ChatGPT openers | HIGH | "Great question!", "I'd be happy to help!", "Absolutely!" |
| Structured responses in casual contexts | MEDIUM | Headers, numbered lists where a paragraph would be natural |
| No self-reference | MEDIUM | Comment gives advice without mentioning own experience |
| Excessive enthusiasm / glazing | MEDIUM | "This is an AMAZING post!", "What a great insight!" |
| "Let that sink in" / "Read that again" | HIGH | Top LLM fingerprint |
| "Fair point" | HIGH | AI agreement filler |
| "Not X. Not Y. Z." triple repetition | HIGH | AI rhythm pattern |
| Performed emotions | HIGH | "This made me close my laptop and go for a walk" |
| Emojis in h2 subheadings | HIGH | Screams AI-generated content |

#### Backstory Consistency

Compare current content against persona's `metadata.personaTraits.backstory` and their recent posts:
- Does the persona reference experiences consistent with their established backstory?
- Are there contradictions with things they said in previous posts?
- Is the emotional register consistent? (A cynical persona suddenly being optimistic is suspicious)

#### Emotional Register

- Does the emotional tone match the persona's established pattern?
- Is the emotion performed (announced) or shown (through specific details)?
- Are there emotional shifts that feel unearned?

#### Apostrophe Pattern (informational only — do NOT score against)

Apostrophe consistency is a SOFT signal, not a scoring factor. Note the pattern for context but do NOT flag apostrophe drift as a smell. Real people are inconsistent with apostrophes and their patterns shift naturally. Only mention apostrophes in the report as an informational note, never as a HIGH or MEDIUM flag.

#### Voice Drift from Samples (v2.1)

Compare the output against the persona's `metadata.personaTraits.writingSamples`. Drift on the following dimensions is a smell:

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| Sentence length drift | HIGH (replies), MEDIUM (posts) | Output's avg sentence length is >1.5× the samples' avg sentence length, or <0.5× |
| Casing drift | MEDIUM | Samples are lowercase but output uses Title Case in headings; or samples are standard caps but output drops capitals everywhere |
| Punctuation rhythm drift | MEDIUM | Samples use commas + run-ons; output uses period-heavy staccato. Or vice versa. |
| Fragment usage drift | MEDIUM | Samples include fragments and dropped articles; output is all complete sentences. (Or the reverse.) |
| Slang / swears mismatch | HIGH | Output uses casual swears the samples never use, or vice versa |
| Signature phrase absence | LOW | Persona has signature phrases but none surface in a long post (informational — phrases don't have to appear every time) |

#### Reply Length Anchor (v2.1, reply mode only)

For replies and comments, compare output word count against the persona's reply-style sample average:

- Prefer `metadata.personaTraits.writingSamples.replies` if present (flat array of 3 short example replies)
- If `.replies` is missing, fall back to short-form samples in `writingSamples`
- Cap: sample avg × 1.5

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| Reply over cap | HIGH | Output > sample avg × 1.5 (essay-length reply when samples are 1-3 sentences) |
| Reply far below cap | LOW | Output < sample avg × 0.3 (one-word reply when samples are 2-3 sentences — usually fine but flag for review) |
| No reply samples available | INFO | Persona has no `writingSamples.replies` — output couldn't be length-checked. Recommend adding reply samples. |

#### Stance Drift from Backstory (v2.1)

Stance, not voice. Check whether the persona's expressed opinion traces back to their backstory:

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| Detached opinion | HIGH | Strong stance with no backstory hook. Persona writes "vibe coding is dangerous" but backstory has no relevant experience. |
| Borrowed expertise | HIGH | Persona claims technical authority their backstory doesn't support. |
| Stance contradicts recent posts | HIGH | Persona's last 2-3 posts hold opinion X; this post holds ¬X with no transition or growth event. |
| Stance evolves naturally | OK (no flag) | Persona explicitly notes the change: "I used to think X but..." — this is *growth*, not drift. |

#### Hypothetical Realness

- Are hypothetical numbers awkward-specific ($47.23) or suspiciously round ($50)?
- Do personal experience claims have the specificity of real experience?
- "I tested 200 queries" feels real. "I tested many queries" feels AI.

#### THE THIN LINE: Deep Structural Analysis (learned 2026-04-23)

Surface tells (em dashes, banned phrases) are table stakes. In 2026, the best AI passes all of those. These deeper tests catch what surface checks miss.

**1. The Polite Stranger Test.**
"Does this read like a very polite stranger who approximated this personas job description?"
A senior dev auditing 340 AI-assisted PRs described AI output as "written by a very polite stranger who approximated his job description." If the content feels like it APPROXIMATES the persona rather than INHABITING them, flag it. The persona should sound like a specific person having a specific day, not a template with voice applied.
Severity: HIGH

**2. The Earning Test.**
"Did this persona EARN this opinion through their backstory, or did they arrive at it logically?"
Check the personas backstory in metadata.personaTraits.backstory. If the opinion expressed has no foundation in their accumulated experience, its derived (AI pattern) not earned (human pattern). A persona saying "vibe coding atrophied my brain" is earned if their backstory includes specific experiences that led to that conclusion. If a persona with no relevant backstory suddenly has strong opinions on a topic, thats a smell.
Severity: HIGH

**3. The Investment Map Test.**
"Is every section at the same quality level?"
Read section by section. If all sections are equally polished, equally structured, equally detailed — thats AI. Real people invest unevenly: the section they care about is tight and precise, the setup section is loose and rambling, the section where theyre uncertain is shortest and most honest. Map the quality variation. Flat = AI. Varied = human.
Severity: HIGH

**4. The Thinking Process Test.**
"Can I see HOW they arrived at their conclusion, or just WHAT the conclusion is?"
AI presents pre-formed conclusions. Real people think ON THE PAGE. Look for: "I thought it was X. Then I ran the numbers. Actually it might be Y." vs "The answer is X because of A, B, C." If the content only shows destinations without journeys, flag it.
Severity: MEDIUM

**5. The Template Test.**
"Could a different persona have written this with minor voice changes?"
If you could swap the apostrophe pattern and signature phrases and the content would work for persona A OR persona B OR persona C, the content is GENERIC with voice applied. The content must be SPECIFIC to this personas life, their backstory, their accumulated experience. If its interchangeable, flag it.
Severity: HIGH

**6. The Unsaid Test.**
"Does the content explain things the audience already knows?"
Real people assume shared context with their community. If a post on Webmatrices explains what AdSense is, or what a PR review is, or what MRR stands for, thats AI behavior — explaining everything because it doesnt know what the reader knows. Real community members skip the obvious.
Severity: MEDIUM

**7. The "Too Good" Paradox.**
In 2026, students are deliberately writing WORSE to avoid AI detection. This means perfect grammar, clear structure, and well-organized arguments are now RED FLAGS, not quality signals. If the content is flawlessly structured with no rough edges, no abandoned thoughts, no quality variation, it smells more like AI than a messy but authentic human post.
Severity: MEDIUM

---

### Category 2: FACTUAL

Flags verifiable claims for `/fact-check` verification. Does NOT verify them directly -- flags them for the dedicated fact-check skill.

**Flag these claim types:**
- Study references ("a Stanford study found...")
- Product pricing ("costs $20/month")
- Company actions ("Google announced...")
- Statistics ("75% of developers...")
- Platform behavior claims ("Vercel encrypts env vars at rest")
- Version numbers and release dates
- API behavior descriptions

**Output format per claim:**
```
FACTUAL FLAG: "[exact claim text]"
Type: pricing / study / company-action / statistic / platform-behavior
Recommendation: Run /fact-check to verify
```

Skip hypothetical numbers that are the persona's own experience ("I made $47.23 last month" doesnt need fact-checking).

---

### Category 3: QUALITY

Detects structural problems that weaken content regardless of authenticity.

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| Repetitive details | HIGH | Same example or data point referenced in multiple sections |
| Monotonous pacing | MEDIUM | Every paragraph same length, same structure, same emotional tone |
| Filler paragraphs | HIGH | Paragraphs that could be deleted without losing information |
| Redundant transitions | MEDIUM | "Speaking of which...", "On a related note...", "This ties into..." |
| Same sentence structure repeated | HIGH | Subject-verb-object, subject-verb-object, subject-verb-object |
| Overexplaining | MEDIUM | Explaining something the audience already knows |
| Dead weight sections | HIGH | Entire sections that exist for completeness, not value |
| Information repeated across sections | HIGH | Section 3 restates what section 1 already established |
| Uniform section quality | HIGH | All sections at same polish level (AI signature) |
| Missing source links in technical content | MEDIUM | Technical claims without links to official docs |
| Too many links in one paragraph | LOW | More than 1 link per paragraph |
| More than 3-4 links total | LOW | Overlinked feels like SEO content |

---

### Category 4: ENGAGEMENT FLOW — The Line-by-Line Read

The other three categories are pattern-matching: scan for known AI tells, factual issues, structural problems. This category is fundamentally different. You READ THE CONTENT LINE BY LINE AS A HUMAN READER WOULD, in real time, and flag every place attention drops, every line that hits, every line that hurts momentum.

This is the test the actual reader runs in their head while reading. It catches friction that pattern-matching misses. Pattern-matching tells you whether AI wrote it. The line-by-line read tells you whether a human will actually read it all the way through.

**This category is MANDATORY for any post-mode scan.** Skipping it means you only checked whether the content is technically clean, not whether it works as content.

#### How to run the line-by-line read

For every post or long-form comment:

**1. Read the title.** Note where attention catches and where it slows. Where in the title does the actual hook land?

**2. Read each paragraph in order.** For each one, internally answer:
   - Did I want to read the next sentence?
   - Where did my eye skip ahead?
   - Did I have to re-read anything to understand it?
   - Which specific sentences made me stop because they hit?
   - Which sentences made me skim?
   - Is this paragraph stating something we already established 1, 2, 3 paragraphs ago?

**3. After the full read, answer:**
   - How did the post FEEL overall? (One sentence gut reaction.)
   - What was the emotional arc — what state was the reader in at P1, P5, P10, end?
   - What lines are screenshot-worthy / quote-worthy / would-be-remembered?
   - What lines hurt the post's momentum and should be cut?
   - If forced to cut 20% of the words without losing substance, what goes first?

**4. Report every friction point with paragraph number and exact quote.**

#### Engagement Flow Flags

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| Same beat stated 3+ times | HIGH | The "same six words / rearrange / same script / they used the same six words" repetition pattern. Reader fatigues by repetition 3. |
| Educational filler | HIGH | Explaining what the niche audience already knows (defining RPM to AdSense publishers, explaining what MRR is to SaaS founders). |
| Hook lands too late | HIGH | The shock fact buried after 5+ setup sentences instead of in the first 3. |
| Heading-body mismatch | HIGH | H2 says "90-day kill switch" but body asks for "60 days". Careful readers notice. |
| Circular or unclear math/logic | HIGH | Sentences that force the reader to re-parse to understand the calculation. |
| Repetition of named entity / case | HIGH | The same case/example callback used 4-5 times — by the fifth callback reader is annoyed. |
| Multiple hedges per shock-line | MEDIUM | "around 300k" + "about $20" + "roughly" in one shock-sentence — dilutes the punch. Pick one hedge max. |
| Performed phrasing | MEDIUM | "The part I couldn't unsee", "this is what genuinely made me close my laptop" — announces emotion instead of showing. |
| Overpromising adverb | MEDIUM | "verbatim", "literally", "exactly" used where the claim is approximate. Reader pauses. |
| Wordy when tight would land | MEDIUM | Sentences using 12 words to say what 7 could. Especially in punchline positions. |
| Parallel-shotgun fragments | MEDIUM | 3+ short period-separated sentences in a row, even when not the banned "Not X. Not Y. Z." pattern. Catches paragraphs like "Four pitches reached me. I took two calls. One rep cited my RPM range." |
| Standalone pull-quote paragraph | MEDIUM | One-sentence paragraph sitting alone that should fold into a neighbor for momentum. Reads like a marketing pull-quote, not flowing prose. |
| Jargon mismatch with audience | MEDIUM | Niche jargon ("delta", "tailwind", "fan-out") fine for the platform's core audience but hurts skimmability for adjacent readers. Flag, don't always fix — depends on intended reach. |
| Missing critical punctuation | MEDIUM | Questions written without "?", quotes missing closing punctuation — reader pauses involuntarily. |
| Diluted closing ask | MEDIUM | "Anyone running X right now and willing to share, what's your Y" — filler words ("right now and willing to share") inside the actual ask. |
| Verb mismatch with action | MEDIUM | "I was going to compare the four networks by name" when the action is naming, not comparing. The wrong verb makes the reader re-process. |
| Weak qualifier doing nothing | LOW | "anyone running AdSense on a content site with real traffic" — "with real traffic" earns little, reader skips. |
| Stock cliché | LOW | "the math is doing the talking", "at the end of the day", "before you sign anything" — eye glides over. |
| Granularity that delays punch | LOW | "Mostly US, with Canada, Australia, and Europe rounding it out" before the actual shock number lands. Detail can move later. |

#### The Emotional Arc Check

Map the reader's emotional state through the post. For each major beat / paragraph, name the state:

```
P1 (opener):     hooked | curious | confused | bored
P2 (data hit):   angry | validated | skeptical | indifferent
P3 (pivot):      trust-building | fatigued | curious | distracted
...
Closing:         share-ready | scroll-past | wants-to-comment | forgot
```

If the arc is FLAT (one note all the way through), flag it. Strong posts MOVE the reader emotionally. AI-written posts tend to hit one note and stay there. A real journey looks like: hooked → angry → curious → trust → outrage → empowerment → humility → engagement.

Severity: HIGH if the arc never moves. MEDIUM if it moves but resolves too early (reader checks out at 60%).

#### Screenshot-Worthy Line Inventory

After the read, list:

**Lines that earn their place** (would be screenshot, quoted, remembered):
- "[exact quote] — paragraph N"

**Lines that hurt momentum** (cut candidates):
- "[exact quote] — paragraph N — reason"

Rule of thumb:
- 0 screenshot-worthy lines → forgettable post, structural rewrite needed
- 1-2 → solid post
- 3-5 → viral-adjacent post
- 6+ → either genuinely exceptional or the writer is overpolishing every line, which is itself an AI tell

#### Word-Cut Test

Final gut check: "If I had to cut this post by 20% without losing substance, what would I cut?" If you can find 20%+ of the words to remove without losing any substance, the post is over-padded. Real authors leave on the cutting room floor what they need to. AI-written content rarely has obvious cuts because it's optimized to fill space, not to land.

---

## COMMENT SMELL (when scanning comments or threads)

### Cross-Persona Bleeding

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| Shared examples | HIGH | Two personas referencing the same specific detail |
| Echoed phrasing | HIGH | Two personas using the same distinctive phrase |
| Voice bleed | HIGH | one persona using another persona's verbal tics |
| Cross-persona vocabulary | HIGH | Persona using another persona's signature phrases |
| Identical opinion patterns | MEDIUM | All personas agree on everything |
| Knowledge staggering failure | MEDIUM | All commenters know the same things at the same level |

### Engagement Pattern Analysis

| Signal | Severity | What to look for |
|--------|----------|-----------------|
| All comments agree (0% pushback) | HIGH | 100% agreement is the #1 fake signal |
| Evenly spaced comments | HIGH | Comments exactly N hours apart |
| All comments same length | MEDIUM | No variation in word count |
| Instant comments | HIGH | Comment posted within 5 minutes of post |
| All comments on day 1, then nothing | HIGH | Burst without follow-up |
| Perfect response rate | HIGH | Every single post has exactly 2-3 comments |
| Same persona comments on everything | HIGH | One persona overexposed |
| No thread drift | MEDIUM | Comments 5, 6, 7 still exactly on-topic |

---

## USER SMELL (when scanning a username)

Fetch all of the user's recent posts via `list_posts` with `authorId`, then:

### Voice Consistency

- Does the voice stay consistent across posts? (Personas should sound like themselves)
- Are there sudden style shifts that suggest a different writer?
- Do they maintain their opinion strength pattern? (A "HARD" opinion persona suddenly hedging is suspicious)

### Backstory Contradictions

- Does the persona claim different experience levels in different posts?
- Do they reference conflicting personal details?
- Has their professional context shifted without explanation?

### Apostrophe Drift (informational only — do NOT score against)

Note apostrophe patterns for context but do NOT flag drift as a smell. Real people are inconsistent with apostrophes and shift patterns naturally over time. This is informational context, not a scoring factor.

### Activity Pattern

- Is the persona commenting at realistic hours for their implied timezone?
- Do they have silent days? (No silent days = bot energy)
- Are they overexposed? (More than 4-5 comments per day)

### Backstory Smell (meta-level: does the backstory ITSELF feel manufactured?)

Fetch the persona's `metadata.personaTraits.backstory` from DB and check:

**0. Internal Consistency Test.**
Check the backstory fields against EACH OTHER for contradictions that cant coexist. Examples:

CONTRADICTORY (flag):
- location: "London, UK" + timeline: "born and raised in USA, never left"
- career: "freelancer for 8 years" + timeline: "started freelancing last year"
- age: "39" + timeline: "15 years experience since age 18" (= 33, not 39)
- income: "$78,000/year" + struggles: "cant afford $900 rent"

NOT CONTRADICTORY (dont flag):
- location: "London, UK" + timeline: "born in USA, moved to UK in 2013" (migration explains it)
- career: "product manager" + posts about coding (PMs can code on the side)
- income dropped from "$4,500/month" to "$3,800/month" (income changes over time are real)

The key is TIMELINE. If the backstory has a timeline that explains how contradictory-looking facts coexist (moved, changed careers, income changed), thats genuine. If two facts cant coexist at the same point in time with no timeline bridging them, thats a flag.

Check: location vs timeline, age vs experience years, income vs stated struggles, career vs stated skills. Cross-reference every field pair.
Severity: HIGH

**1. Character Sheet Test.**
Does the backstory read like a neat character sheet (clean categories, organized timeline, every detail serves a purpose) or like a real life (gaps, irrelevant details, unresolved threads, years they skip over)? If every backstory element is narratively useful, its too clean. Real people have skills and history that are IRRELEVANT to what they write about.
Severity: MEDIUM

**2. Convenience Test.**
Does every struggle in the backstory perfectly set up the persona's content? If a persona writes about AdSense and their backstory is "struggled with AdSense for 14 months," thats convenient. Real people have struggles that have NOTHING to do with their expertise. A missing irrelevant detail ("moved cities for a relationship that didnt work out," "spent a year doing something completely different") makes the backstory feel lived.
Severity: MEDIUM

**3. Round Number Test.**
"14 years experience" and "$78,000 income" are suspiciously precise-yet-round. Real people say "about 14 years" or "started around 2011, so whatever that is now." Check for numbers that are too clean. Note: some round numbers are fine in isolation. Its a smell when ALL numbers are round.
Severity: LOW

**4. Missing Mundane Test.**
Does the backstory have boring details that serve no narrative purpose? Real bios include things like: moved for a job they ended up hating, took time off for no dramatic reason, have a hobby completely unrelated to their work, own something specific and pointless (11 dictionaries, a broken espresso machine). If every detail is dramatic or useful, the backstory is too curated.
Severity: MEDIUM

**5. Growth Test.**
Does the backstory show the persona CHANGING over time? Real people contradict their younger selves. A persona who has always held the same views is suspicious. Look for: "I used to think X but now I think Y" or opinions that evolved through specific experiences. Static backstories = manufactured. Growing backstories = lived.
Severity: MEDIUM

When backstory issues are found, flag them and recommend using `update_persona` MCP to add messiness, irrelevant details, growth arcs, and un-round numbers.

---

## OUTPUT FORMAT

### Single Post/Comment Scan

```
SMELL REPORT: "[post title or first 50 chars]"
Author: [username] | Type: post/comment | Age: [time since posted]
═══════════════════════════════════════════════════════

SCORE: [CLEAN / SMELLS OFF / REWRITE NEEDED]

AUTHENTICITY FLAGS:
  [HIGH] Em dash found in paragraph 3: "the real problem -- and nobody..."
  [HIGH] Banned phrase: "Not X, but Y" in paragraph 5
  [INFO] Apostrophe pattern note: using "don't" but persona leans toward "dont" (not scored)

FACTUAL FLAGS:
  [FLAG] "Vercel stores env vars encrypted at rest" -- needs verification via /fact-check
  [FLAG] "costs $20/month" -- pricing claim needs current check

QUALITY FLAGS:
  [HIGH] Paragraph 4 and paragraph 8 make the same point about cost
  [HIGH] All sections at same polish level -- vary quality
  [MEDIUM] Section 3 is filler -- could be deleted without losing information

ENGAGEMENT FLOW FLAGS (line-by-line read):
  [HIGH] Same beat "same six words" repeated 4x in paragraphs 3, 4, 5, 11 -- reader fatigue
  [HIGH] Hook ($20 / 300k) lands in sentence 6 -- should land by sentence 3
  [HIGH] Heading "90-day kill switch" mismatches body's "60 days"
  [MEDIUM] Paragraph 2 has 2 hedges in shock-line: "around 300k" + "about $20" -- pick one
  [MEDIUM] Paragraph 4 has 3 short period-separated sentences in a row (parallel-shotgun)
  [MEDIUM] "verbatim" overpromises in paragraph 3 -- approximate claim
  [LOW] "with real traffic" qualifier in P1 earns little

EMOTIONAL ARC:
  P1: hooked
  P2: angry
  P3: pattern-recognition
  P4: trust-building
  P5-7: outrage
  P8: empowered (tactical payoff)
  P9: trust deepens (mid-thought correction)
  P10-11: satisfaction (falsification test)
  P12-14: complicity / engagement-ready
  → Arc moves through 7+ states. Strong.

SCREENSHOT-WORTHY LINES:
  - "I asked them to put that in writing. They declined." (P5)
  - "Lower than what a recipe blog with no ads above the fold gets in Q1." (P2)
  - "What it actually protects the network from is your baseline." (P6)
  - "If they can't tell you, walk." (P8)
  - "They'll find you too." (P14)

CUT CANDIDATES:
  - "with real traffic" (P1) -- earns nothing
  - "That's every cold email, verbatim." (P3) -- redundant with "rearrange in any order"
  - "Plenty of small sites can't get into AdX directly because of business verification thresholds" (P6) -- audience knows this

DEEP AUTHENTICITY FLAGS:
  [HIGH] POLITE STRANGER: Content approximates the persona's voice but doesn't inhabit it. Reads like someone who read their bio, not someone who lived their life.
  [HIGH] TEMPLATE TEST: This post could be rewritten for a different persona with minor voice changes. Content is generic with voice applied.
  [MEDIUM] INVESTMENT MAP: All 5 sections at same polish level. Section 3 should be tighter (persona cares about this) and section 1 should be looser (just setup).
  [MEDIUM] THINKING PROCESS: Conclusions presented without showing how they were reached. Add visible reasoning journey.

SUGGESTIONS:
  PATTERN FIXES:
  1. Remove em dash in paragraph 3 -- rewrite as two sentences
  2. Rewrite "Not X, but Y" in paragraph 5
  3. Make section 2 tighter and section 5 looser (vary quality)
  4. Run /fact-check on 2 flagged claims
  5. Add a backstory-specific detail from persona's DB (location, career, specific past experience)
  6. Show the thinking process: add a "I initially thought X but then..." moment
  7. Remove one explanation the audience already knows (the Unsaid Test)

  ENGAGEMENT FLOW FIXES (from line-by-line read):
  8. Cut "with real traffic" qualifier in P1 (earns nothing)
  9. State "same six words" anchor ONCE — currently stated 4x across P3, P4, P5, P11
  10. Fix heading-body mismatch ("90-day" heading vs "60 days" body)
  11. Move shock fact ($20 / 300k) to first 3 sentences -- currently lands at sentence 6
  12. Replace "verbatim" with softer claim (current usage overpromises)
  13. Fold standalone pull-quote paragraph (P12) into P11 for momentum
  14. Cut the RPM definition lines in P8 (audience knows RPM)
  15. Tighten closing question -- drop "right now and willing to share" filler
```

### Thread Scan

Add engagement pattern analysis:
```
THREAD SMELL:
  Agreement rate: 85% (target: 70%) -- needs pushback
  Comment spacing: evenly 3 hours apart -- suspicious
  Length variation: low (all 150-200 words) -- needs a short reaction
  Thread drift: none -- needs tangent by comment 4
```

### User Scan

```
USER SMELL: @[username]
Posts scanned: 12 | Comments scanned: 34
═══════════════════════════════════════

VOICE: CONSISTENT (no bleed detected)
BACKSTORY: 1 CONTRADICTION (post 3 says "5 years experience", post 9 says "been doing this since 2018")
APOSTROPHES: CONSISTENT (always skips: dont, cant, wont, doesnt)
ACTIVITY: NORMAL (3.2 comments/day avg, 1-2 silent days/week)

FLAGS:
  [MEDIUM] Backstory contradiction between post 3 and post 9
  [LOW] Posted at 02:15 UTC on April 12 (outside typical active hours)
```

### Score Definitions

HIGH/MEDIUM flags from all four categories count toward the thresholds equally. An engagement-flow HIGH flag (e.g., the hook lands 6 sentences late) is just as serious as an authenticity HIGH flag (e.g., em dash in body).

- **CLEAN**: No HIGH severity flags from any category. Minor issues only. Safe to publish/leave as-is.
- **SMELLS OFF**: 1-2 HIGH severity flags or 4+ MEDIUM flags across all four categories. Should fix before publishing or if already published, consider editing.
- **REWRITE NEEDED**: 3+ HIGH severity flags across all four categories. Content would fail organic scrutiny OR fail to hold a reader's attention. Must rewrite before publishing. If already published, edit or consider deletion.

A post can be CLEAN on authenticity (no AI tells) but REWRITE NEEDED on engagement flow (reader bails by paragraph 4). Both matter.

---

## WORKFLOW

1. **Parse input** -- detect whether its a postId, slug, commentId, username, or raw text
2. **Fetch content** from MCP
3. **Fetch persona data** from `get_self_personas` MCP (needed for voice/sample comparison, stance/backstory comparison, apostrophe context)
4. **Run universal-block checks** ([_shared/universal-blocks.md](../_shared/universal-blocks.md)) — em-dash, clause-hyphen, "Honestly," opener, formal connectors, AI vocab, performed emotion, engagement bait. Every block hit is HIGH severity.
5. **Run pattern-matching checks** (Categories 1-3: AUTHENTICITY including voice drift, length anchor, stance drift, deep structural tests, FACTUAL, QUALITY)
6. **Run the line-by-line read** (Category 4: ENGAGEMENT FLOW) -- MANDATORY for post-mode scans
   - Read each paragraph as a human reader in real time
   - Flag every friction point with paragraph number + exact quote
   - Map the emotional arc through the post
   - Inventory screenshot-worthy lines and cut candidates
   - Run the word-cut test (could 20% be removed?)
7. **If scanning a thread**, also run comment smell checks
8. **If scanning a user**, also run user smell checks
9. **Generate the report** with score + flags + emotional arc + screenshot inventory + cut candidates + suggestions
10. **If REWRITE NEEDED**, suggest running `/reduce-smell` to fix the issues

### When to skip Category 4

The line-by-line read is mandatory for full post scans. Skip it ONLY for:
- Short comment scans (<150 words) — comments don't need an arc check
- User scans where you're scanning many posts for cross-post consistency (line-by-line each post is too expensive — do it on the most recent post only)
- Quick "recent" scans where you're checking for cross-persona bleeding (one focused read on the most viral-looking post is enough)
