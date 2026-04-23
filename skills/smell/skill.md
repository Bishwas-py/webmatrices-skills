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

#### Apostrophe Pattern

- Fetch persona's `writingSamples` from MCP traits
- Compare apostrophe usage in current content against their established pattern
- Flag drift: if they usually skip apostrophes in "dont/cant/wont" but this content uses them consistently, thats a smell

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
Check the personas backstory in metadata.personaTraits.backstory. If the opinion expressed has no foundation in their accumulated experience, its derived (AI pattern) not earned (human pattern). nolanriggs saying "vibe coding atrophied my brain" is earned — he built 4 apps with $0 revenue and couldnt write a sorting function. If a persona with no relevant backstory suddenly has strong opinions on a topic, thats a smell.
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
If you could swap the apostrophe pattern and signature phrases and the content would work for romanking OR nolanriggs OR digitaldave01, the content is GENERIC with voice applied. The content must be SPECIFIC to this personas life, their backstory, their accumulated experience. If its interchangeable, flag it.
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
| Voice bleed | HIGH | digitaldave01 using techwizardrino's verbal tics |
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

### Apostrophe Drift

- Compare apostrophe patterns across posts over time
- Real people are consistently inconsistent. They ALWAYS skip apostrophes in the same words.
- If apostrophe usage changes dramatically between posts, thats a smell.

### Activity Pattern

- Is the persona commenting at realistic hours for their implied timezone?
- Do they have silent days? (No silent days = bot energy)
- Are they overexposed? (More than 4-5 comments per day)

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
  [MEDIUM] Apostrophe pattern inconsistent with persona DB (using "don't" but persona pattern shows "dont")

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
  3. Match apostrophe pattern to persona's established usage
  4. Make section 2 tighter and section 5 looser (vary quality)
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
USER SMELL: @techwizardrino
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
