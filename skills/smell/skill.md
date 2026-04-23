---
name: smell
description: Universal authenticity scanner for Webmatrices content. Scans posts, comments, user profiles, or raw text for AI tells, factual issues, quality problems, and cross-persona bleeding. Use when asked to audit content, check authenticity, detect AI writing, review engagement quality, or verify organic appearance.
disable-model-invocation: true
argument-hint: [postId/slug/commentId/username/"raw text"]
---

# Smell

Universal authenticity scanner. Detects AI tells, factual issues, quality problems, and cross-persona bleeding in any Webmatrices content. This skill absorbs the old detect-fake-engagement functionality and adds deeper content analysis.

No subcommands. Auto-detects what you're scanning based on the input.

For engagement psychology research, see [engagement-psychology.md](../_shared/engagement-psychology.md).
For reply quality patterns, see [reply-patterns.md](../_shared/reply-patterns.md).

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

## THREE SMELL CATEGORIES

### Category 1: AUTHENTICITY

Detects patterns that make content feel AI-generated or fake.

#### AI Voice Tells

| Signal | Severity | Example |
|--------|----------|---------|
| Em dash (--) in content | HIGH | "the real problem -- and nobody talks about this -- is..." |
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

DEEP AUTHENTICITY FLAGS:
  [HIGH] POLITE STRANGER: Content approximates the persona's voice but doesn't inhabit it. Reads like someone who read their bio, not someone who lived their life.
  [HIGH] TEMPLATE TEST: This post could be rewritten for a different persona with minor voice changes. Content is generic with voice applied.
  [MEDIUM] INVESTMENT MAP: All 5 sections at same polish level. Section 3 should be tighter (persona cares about this) and section 1 should be looser (just setup).
  [MEDIUM] THINKING PROCESS: Conclusions presented without showing how they were reached. Add visible reasoning journey.

SUGGESTIONS:
  1. Remove em dash in paragraph 3 -- rewrite as two sentences
  2. Rewrite "Not X, but Y" in paragraph 5
  3. Make section 2 tighter and section 5 looser (vary quality)
  5. Run /fact-check on 2 flagged claims
  6. Add a backstory-specific detail from persona's DB (location, career, specific past experience)
  7. Show the thinking process: add a "I initially thought X but then..." moment
  8. Remove one explanation the audience already knows (the Unsaid Test)
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

- **CLEAN**: No HIGH severity flags. Minor issues only. Safe to publish/leave as-is.
- **SMELLS OFF**: 1-2 HIGH severity flags or 4+ MEDIUM flags. Should fix before publishing or if already published, consider editing.
- **REWRITE NEEDED**: 3+ HIGH severity flags. Content would fail organic scrutiny. Must rewrite before publishing. If already published, edit or consider deletion.

---

## WORKFLOW

1. **Parse input** -- detect whether its a postId, slug, commentId, username, or raw text
2. **Fetch content** from MCP
3. **Fetch persona data** from `get_self_personas` MCP (needed for voice/apostrophe comparison)
4. **Run all three categories** of checks (AUTHENTICITY including deep structural tests, FACTUAL, QUALITY)
5. **If scanning a thread**, also run comment smell checks
6. **If scanning a user**, also run user smell checks
7. **Generate the report** with score + flags + suggestions
8. **If REWRITE NEEDED**, suggest running `/reduce-smell` to fix the issues
