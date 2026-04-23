---
name: write
description: Write and publish content for Webmatrices. Handles posts, replies, comments, and HN replies. Fetches persona data from MCP, researches via SuperMCP, enforces authenticity rules, fact-checks claims, previews via Playwright, and publishes via MCP. Use when asked to write a post, draft content, reply to a post, write a comment, write an HN reply, or publish content.
disable-model-invocation: true
argument-hint: [persona] [topic / reply postId / hn-reply "text"]
---

# Write

Write viral, engagement-ready content for Webmatrices and publish it. This skill handles voice, structure, authenticity, publishing via MCP, and designing content that naturally attracts organic engagement. It also handles replies, comments, and HN responses.

For topic discovery, use `/trending` first, then hand off here.
For persona matching, see the [audience-matcher skill](../audience-matcher/SKILL.md).
For engagement psychology and organic patterns, see [engagement-psychology.md](../_shared/engagement-psychology.md).
For reply quality patterns, see [reply-patterns.md](../_shared/reply-patterns.md).

## Subcommands

| Command | What it does |
|---------|-------------|
| `/write [persona] [topic]` | Draft a post as that persona about that topic |
| `/write [persona] reply [postId]` | Draft a comment on an existing post |
| `/write [persona] hn-reply "[text]"` | Draft a Hacker News reply to quoted text |
| `/write preview` | Preview the current draft via `preview_post` or `preview_comment` MCP + open in Chrome via Playwright |
| `/write publish` | Publish the approved draft via `create_post` or `create_comment` MCP + `generate_og_image` |

If no persona provided, auto-match using audience-matcher logic after fetching personas from MCP.
If no args at all, ask what to write about.

---

## PERSONA DATA — ALWAYS FROM MCP

**NEVER hardcode persona usernames, IDs, voice descriptions, or backstories.**

At the start of every /write invocation:

1. Call `get_self_personas` MCP tool to fetch all personas with their metadata
2. Each persona object includes: `id`, `username`, `metadata.personaTraits` (voice, topics, opinion strength, backstory, writing samples, apostrophe patterns)
3. If a persona name is provided as argument, match it against the fetched list
4. If no persona specified, use the persona traits to auto-match against the topic

### Voice Continuity

After selecting a persona, fetch their last 2-3 posts using `list_posts` with `authorId` to:
- Match their recent voice and tone
- Avoid repeating topics they recently covered
- Check backstory consistency (dont contradict something they said last week)
- Match their apostrophe patterns from the DB `metadata.personaTraits.writingSamples`

---

## THE FORMULA

```
BUZZWORD + RESEARCH + LIFE + PERSONALITY = VIRAL CONTENT
```

What separates 6,000 readers from 400:
`external validation + original data + universal pain + controversy + timing`

---

## RESEARCH VIA SUPERMCP

Before writing, gather current data using SuperMCP tools. Run these in parallel where possible:

| Tool | When to use |
|------|-------------|
| `reddit_search` | Community sentiment, pain points, real stories |
| `twitter_search` | Hot takes, breaking reactions (max 1-2 word queries) |
| `devto_trending` | Developer content trends |
| `medium_tag_feed` | Long-form content trends by topic |
| `news_search` | Current events, announcements, data |

Extract: specific numbers, pain points, contrarian angles, universal frustrations, verifiable claims.

---

## HARD BANNED LIST

Check EVERY draft against this list. Violating any of these kills authenticity.

| BANNED | WHY |
|--------|-----|
| Em dash (--) | #1 AI signature tell. EXCEPTION: the admin persona's help replies only |
| Colons in titles | Robotic |
| Colons in prose | Stops the flow |
| "Not X, but Y" structures | Claude's fingerprint |
| "Let's dive in" | LinkedIn-ism |
| "Here's the thing" | AI filler |
| "It's worth noting" | Academic AI voice |
| "Interestingly" | Padding |
| "Moreover", "crucial", "delve", "leverage" | AI vocabulary |
| "In conclusion" / "In summary" | Essay structure |
| "What do you think?" | Engagement bait |
| Bullet points in prose | Use sentences |
| Perfect punctuation throughout | Too polished = suspicious |
| Two clever metaphors per post | One max |
| "I'd be happy to" | AI slop |
| Excessive ellipsis | AI rhythm tell |
| Glazing / excessive praise | AI pattern |
| Repeating the same point | AI padding |
| "r/subreddit" as source | Curation, not ownership |
| Upvote/comment counts | Reporting, not analysis |
| "trending on Reddit/HN/Twitter" | Aggregator energy |
| "Let that sink in" / "Read that again" | Top LLM fingerprint, caught on HN immediately |
| "Let me get this straight" | Over-constructed opening, AI scaffolding |
| "I'll say it louder" / "Say it louder for the people in the back" | Performative AI emphasis |
| "Not X. Not Y. Z." triple repetition | AI rhythm pattern, instantly detectable |
| "This is the part that made me [dramatic reaction]" | Performed emotion, AI loves this |
| Uniform paragraph quality throughout | AI writes at even quality. Humans write some sections tight and others loose |
| "Someone said" / "One person pointed out" / "Another commenter" | Source laundering. Persona must own all insights. See Ownership Principle. |
| "Fair point" | AI agreement filler, especially deadly in HN replies |
| "Comprehensive" / "robust" / "streamline" | Corporate AI vocabulary |
| "It goes without saying" | If it goes without saying, dont say it |
| "At the end of the day" | Cliche filler |
| "Game-changer" / "paradigm shift" | Buzzword AI vocabulary |
| "Needless to say" | AI padding |
| "In my humble opinion" | Nobody actually says this |

### Subheading Rules

- ALL subheadings must be `<h2>`. Never `<h3>` or deeper in posts.
- ZERO emojis in `<h2>` tags. Ever. Emojis in headings scream AI-generated content.
- Subheadings should be lowercase, punchy, and specific. Not "Understanding the Problem" but "the $218 question nobody asks".

---

## AUTHENTICITY MARKERS

### Apostrophe Psychology

Real people are inconsistent. Match the persona's pattern from `metadata.personaTraits.writingSamples`:

- Skip apostrophes in very common contractions: `dont, cant, wont, theres, didnt, wasnt, Ive`
- Keep them in less common ones: `she'd, they're, wouldn't`
- Casual run-ons = fewer apostrophes
- More formal sentences = more apostrophes
- Check the persona's actual posts in DB to match their specific pattern

### Typos (use sparingly, 1-2 max per post)

- Transposed letters: "teh", "hte", "adn", "taht"
- Missing small words: forgetting "a", "the", "to" mid-sentence
- Middle letters drop first in long words

### Capitalization

- Proper nouns always capitalized even in lowercase posts: ChatGPT, Reddit, Claude
- ONE emphatic word in caps sparingly: "this is NOT about ranking"
- Ironic caps for mockery: "Very Serious Business"
- Strategic lowercase in emotional moments

### Rhythm Fix

- Staccato shotgunning (period. period. period.) without breathing room = robot
- Mix short punches with compound sentences
- Every 3-4 short sentences, one longer flowing one

### The Imperfection Principle

Perfect = suspicious. Real people dont proofread Reddit posts three times. One or two unnoticed errors max. More than that looks fake.

### AI-Detection Resistance (learned from HN, 2026-04-21)

HN readers specifically detected AI prose even with lowercase/apostrophe tricks applied. The tells werent individual phrases but structural patterns:

1. **Uniform quality across sections.** AI writes every paragraph at the same polish level. Real humans write some sections tight (the part they care about) and others loose (the part they rushed). Deliberately vary quality across sections.

2. **Performed emotions.** "This is the part that genuinely made me close my laptop and go for a walk" reads as AI because the emotion is announced, not shown. Instead use specific mundane details: "I read the report on my phone at 1am and couldnt sleep after."

3. **No mid-thought corrections.** AI never backtracks within its own text. Real people do. "Actually wait, I looked at this more and..." or "Okay I need to correct something I got wrong in my initial read" is deeply human and AI never generates it unprompted.

4. **No abandoned tangents.** Real writers start thoughts they dont finish. "I started looking into how many YC companies have had security incidents tied to... actually thats a different rabbit hole for a different post." AI always completes every thought.

5. **Too-smooth transitions.** Every section flows perfectly into the next. Real posts have abrupt section shifts where the writer clearly started a new thought without connecting it.

6. **Missing source links.** Real technical writers link to their sources. AI summaries dont. Include 2-3 relevant links in technical posts (official docs, incident pages, research reports). This was specifically flagged on HN.

### Cross-Persona Consistency Check (learned 2026-04-21)

When multiple personas comment on the same post, check for accidental overlap:

- **No shared examples.** If persona A mentions "a meeting summarizer I used twice in January" in the post, persona B cannot use the same example in the comments. Different personas must reference different specific details.
- **No echoed phrasing.** Two personas should never use the same distinctive phrase or framing. If the post says "the value proposition IS the access," a commenter cant say "the value prop is the access."
- **Stagger knowledge levels.** Not everyone knows the same things. If the post corrects a misconception, a commenter written BEFORE reading the post can still hold the old misconception. Thats realistic.

### Varied Section Quality (CRITICAL)

Deliberately make some sections of the post tighter and more polished than others. The part the persona cares most about should be the best-written. The "I should probably mention this too" section should feel rushed, shorter, less refined. This is the single biggest differentiator between AI-written and human-written long-form content.

### THE THIN LINE: Deep Authenticity (learned 2026-04-23)

Surface-level checks (em dashes, banned phrases, apostrophes) are table stakes. In 2026, the best AI passes all of those. A professor confirmed: "ChatGPT 5.1 has no giveaways. No em-dashes. It just sounds like normal, high-quality writing." The real fight is structural.

**The Polite Stranger Problem.** A senior dev auditing 340 AI-assisted PRs said: "His PRs look like they were written by a very polite stranger who approximated his job description." This is exactly what bad persona writing looks like. A polite stranger who approximated what a persona would say. The persona must not approximate. It must INHABIT.

**Five tests every draft must pass:**

1. **SPECIFICITY FROM BACKSTORY.** Every hypothetical detail must come from the personas accumulated life story in metadata.personaTraits.backstory. Not generic. Not plausible-sounding. SPECIFIC to THIS persons life. The persona doesnt mention "a cafe" -- they mention the specific cafe from their backstory with the exact price change. They dont say "I lost my job" -- they reference the specific detail from their backstory that only they would know.

2. **EARNED OPINIONS.** The persona doesnt conclude X because its logical. They conclude X because their backstory led them there through accumulated frustration or joy. The persona doesnt say "vibe coding has costs" because a study showed it. They say it because THEY lived through the specific experience in their backstory that earned them that view. The opinion must feel ARRIVED AT, not DERIVED.

3. **EMOTIONAL INVESTMENT MAPPING.** The section they care about most gets the tightest writing. The setup section is looser, maybe rambling. The section where theyre uncertain is shortest and most honest. AI writes everything at equal quality. Map the personas emotional investment to writing quality per section.

4. **VISIBLE THINKING PROCESS.** Not: "The answer is X because of A, B, C." But: "I thought it was X. Then I ran the numbers. Actually it might be Y. Im not sure anymore." Show the JOURNEY of the thought, not just the destination. Real people think ON THE PAGE. AI presents pre-formed conclusions.

5. **THINGS LEFT UNSAID.** Real people dont explain everything. They assume shared context. They leave things implied. They skip the obvious. If the persona is writing for a community that already understands AdSense, they dont explain what AdSense is. AI explains everything because it doesnt know what the reader already knows.

**The Template Test (final gut check before publishing):** Could a different persona have written this with minor voice changes? If yes, the content is GENERIC with voice applied. Kill it. The content must be SPECIFIC to this personas life, not interchangeable.

**What students and writers taught us:** In 2026, students are deliberately writing WORSE to avoid AI detection — diluting vocabulary, breaking flow, inserting errors. Good human writing now looks like AI. This means our goal is not "write perfectly with human markers sprinkled in." Our goal is to write like a SPECIFIC PERSON with a SPECIFIC HISTORY would write on THIS topic on THIS day. Sometimes that means messy. Sometimes that means brilliant. Always it means particular.

---

## OWNERSHIP PRINCIPLE

The Core Test: "Did I find it or figure it out?"
- Found = curating (weak)
- Figured out = owning (strong)

The Deeper Test: "Is this persona talking about THEIR life or reporting on SOMEONE ELSE's life?"
- Their life = authentic (strong)
- Someone else's life = journalism (kill it)

**OWN without attribution:** Insights from anonymous tweets/comments, patterns you synthesized, frameworks you created, "everyone knows but nobody says" truths, emotional observations.

**ATTRIBUTE:** Academic studies, named sources with specific data, direct quotes from named people, statistics from published reports (FRED, Google, etc).

**Name your patterns.** Once named, it's yours:
- "The 12x cost problem"
- "The $799 stack"
- "The Control Tax"
- "The Success Theater Economy"

You are NOT a curator, summarizer, reporter, or aggregator.
You ARE an analyst, synthesizer, storyteller. The person with the take.

### The Retelling Trap (CRITICAL)

The most common failure mode. You find a juicy Reddit story and retell it with commentary. This is JOURNALISM, not content creation. Every persona speaks from THEIR experience.

**NEVER write posts that are structured as:**
- "Someone did X. Here's what happened. Here's what I think about it."
- "A guy in [country] got [consequence] for [action]. The reaction was [split/mixed/heated]."
- "Someone posted a theory that [take]. And they're not wrong."
- "[N] people agreed with this. The comments are [adjective]."

**INSTEAD, internalize the theme and make it the personas own story:**
- Reddit story: "dev got fired for automating his job" -> the persona writes about THEIR fear that they're automating too much of THEIR own workflow and where the line is
- Reddit thread: "astroturfing accusations about Claude Code" -> the persona writes about the suspicious pattern THEY keep seeing in dev forums and why the panic narrative doesnt match the hiring data THEY looked up
- Reddit discussion: "TurboQuant is revolutionary" -> the persona writes about the paper THEY spent two days reading and what THEY think it means

The persona lived it, noticed it, thought about it, has an opinion about it. They didnt read about it on the internet and decide to share it with their community.

### Source Laundering Rule (CRITICAL)

Reddit, Twitter, HN are RESEARCH INPUTS, never content. The reader must never know where the idea came from.

**NEVER do this:**
- "2,462 people upvoted a post on r/cscareerquestions saying..."
- "A thread on r/programming revealed..."
- "Reddit users are reporting that..."
- "Someone on r/artificial shared a benchmark showing..."
- "Someone posted a theory that..."
- "The internet split on this one..."
- "[N] people agreed with this"
- "The comments are [chaos/heated/split]"
- Mention specific subreddit names as sources
- Cite upvote counts or comment counts
- Reference "this week on Reddit" or "trending on HN"
- Retell someone else's story as the framing of your post

**DO this instead:**
- Absorb the theme, throw away the story wrapper entirely
- The persona encountered this idea through their OWN work, not through scrolling
- "Ive been automating about 30% of my workflow and lately im wondering where the line is" (not "a guy got fired for automating his job")
- "I keep seeing the exact same post format in every dev forum and its creeping me out" (not "someone accused Claude Code of astroturfing")
- "I spent two days reading this paper and I think it changes everything" (not "this paper dropped and people are excited")

Reddit gives you the THEME. The persona provides the STORY. Those are two completely different things.

---

## ENGAGEMENT-READY CONTENT DESIGN

Every post written by this skill should be designed to naturally attract comments. This connects /write output with `/simulate-engagement`.

For the full psychology behind these patterns, see [engagement-psychology.md](../_shared/engagement-psychology.md).

### The Gap Principle

Dont write complete, exhaustive posts. Leave deliberate openings for commenters:

- **One missing angle** -- cover 3 of 4 perspectives, leave the 4th for a commenter to supply
- **One debatable claim** -- include a take thats defensible but not bulletproof. Correction comments are 3-4x longer than agreement comments.
- **Specificity that invites comparison** -- "my site gets 1,200 sessions/month mostly from Pinterest" invites others to share their own numbers
- **End with a genuine question** -- not engagement bait ("what do you think?") but a real question the persona actually wants answered: "anyone else seeing this pattern or is it just my niche?"

### Engagement Hooks by Type

| Hook | What it attracts | Example |
|------|-----------------|---------|
| **Vulnerable question** | Helpers, lurkers breaking silence | "am i the only one whose adsense rpm tanked after the march update?" |
| **Specific personal number** | People comparing their own data | "i made $47.23 last month from 1,200 sessions" |
| **Slightly wrong take** | Correctors (longest, most detailed comments) | "honestly i think backlinks are dead for small sites" |
| **Narrow situation** | People in the exact same boat | "8-month-old blog, 23 posts, stuck at 500 sessions" |
| **Named pattern** | Status signalers wanting to add their own framework | "i call it the Control Tax" |
| **Unpopular opinion** | Second dissenters who felt alone | "vibe coding made me a worse developer" |

### Hypothetical Numbers

When the persona references their own experience with numbers, use awkward-specific numbers, NEVER round:
- YES: $47.23, 1,247 sessions, 23 posts, 6.2 seconds, $218/month
- NO: $50, 1,000 sessions, 20 posts, 5 seconds, $200/month

Round numbers scream "made up." Awkward numbers scream "I actually looked at my dashboard."

### The Engagement Blueprint

When presenting a draft, also output an **engagement blueprint**:

```
ENGAGEMENT BLUEPRINT:
- Trigger type: [correction / recognition / status-signal / emotional-vent]
- Gap left: [specific angle not covered]
- Debatable claim: [the intentionally imperfect take, paragraph number]
- Suggested first comment: [persona] [type: agree+add / disagree / tangent / question] (~Xmin after post)
- Suggested second comment: [persona] [type] (~Xhrs after first)
- Suggested tangent: [persona] [adjacent topic] (next day)
- Distribution: 70% agree / 20% pushback / 10% tangent
```

### Link Rules

- Maximum 3-4 links total per post
- Maximum 1 link per paragraph
- Links must be to primary sources (official docs, papers, announcements), not blog posts about the source
- No bare URLs. Always anchor text.
- Technical posts need at least 2-3 source links (HN flagged missing links as AI tell)

---

## HOOK ENGINEERING

| Hook Type | Example | Why It Works |
|-----------|---------|--------------|
| Paradox | "75% use it. 40% trust it." | Cognitive dissonance |
| Recognition | Everyone recognizes this pattern | Reader feels seen |
| Shared Enemy | Unite against a pattern, not a person | Safe anger |
| Uncomfortable Truth | "you're training your replacement. for free." | Says what everyone thinks |
| Identity Threat | "your voice isn't good enough" | Deep fear |
| Death Announcement | "Stack Overflow is dead." | Urgency + controversy |
| Countdown | "12 months. That's all you get." | Pressure creates motion |
| Karma Flip | "X did Y for 15 years. Now look." | Satisfying irony |

---

## TITLE RULES

- Lowercase, punchy, contains tension or paradox
- No colons. Ever.
- No question marks (statements outperform questions)
- Specific number beats vague claim: $218, 12x, 90%
- Named victim or threat beats abstract concept
- Built-in conflict beats explanation

**5-second test before finalizing title:**
1. Is there a specific number?
2. Is there a human victim or threat?
3. Is there built-in conflict?
4. Would someone screenshot a single line from this?

---

## EMOTIONAL PACING

```
exhausted -> amused -> angry -> thoughtful -> sad -> motivated
```

Dont flatline. The reader's emotion should move. Each section shifts the mood slightly. End on motivated or resigned. Never on happy or promotional.

---

## STRUCTURE RULES

- Each section needs a hook that forces reading the next section
- Data anchors credibility. Use sparingly, high impact.
- Shorter paragraphs, more whitespace
- Hooks at every section start, not just the opening
- Remove scaffolding: anything that feels like setup without payoff, cut it
- Statements at the end. Not questions. Quiet confidence.
- Format as HTML: `<p>`, `<h2>`, `<strong>`, `<ol>`, `<ul>`, `<blockquote>`, `<a>`

---

## WORKFLOW: POST MODE

### Step 1: Fetch personas and determine author

1. Call `get_self_personas` to fetch all personas with their traits
2. If persona provided as argument, match against fetched list
3. If no persona, auto-match topic against persona traits (topics, voice, backstory relevance)
4. Consider persona fatigue: if same persona posted in last 3 days, deprioritize unless topic is a perfect fit
5. Fetch persona's last 2-3 posts via `list_posts` for voice continuity

### Step 2: Research (if needed)

If the topic needs current data, run SuperMCP searches in parallel:
1. `reddit_search` for community sentiment and pain points
2. `twitter_search` for hot takes (max 1-2 word queries)
3. `news_search` for current events and data
4. `devto_trending` or `medium_tag_feed` for content landscape
5. Extract specific numbers, quotes, pain points

### Step 3: Write the draft

1. **Title** -- apply title rules, no colons, lowercase, specific number if possible
2. **Opening line** -- hook immediately. Fact or tension. No context-setting.
3. **Body** -- follow emotional pacing, mix paragraph lengths, one metaphor max
4. **Closing** -- statement, not question. Unless persona demands a question.
5. **Format** -- HTML: `<p>`, `<h2>`, `<strong>`, `<ol>`, `<ul>`, `<blockquote>`
6. **Engagement design** -- apply the Gap Principle. Leave one angle uncovered, include one debatable claim, end with genuine question.

### Step 3.5: Fact-check (MANDATORY)

Auto-run `/fact-check` internally on every verifiable claim before presenting the draft.

Every technical claim, statistic, and product behavior description must be verified. This step exists because HN tore apart a post (2026-04-21) for getting Vercel's encryption model wrong.

**Verify against primary sources first:**
1. Official docs (e.g., vercel.com/docs, not a blog post ABOUT vercel)
2. Official incident pages / security bulletins
3. Published research papers or reports from named organizations

**Watch for these traps:**
- Third-party blogs often misrepresent official docs
- "Encrypted at rest" means different things in different contexts. Be precise.
- HN/Reddit comments correcting posts are sometimes wrong too. Verify corrections against primary sources.

**If unsure about a technical claim:**
- Reframe as the persona's initial understanding + self-correction. "My first reaction was X but after reading the docs its actually Y."
- Or remove the claim entirely. Spicy opinions are fine. Wrong facts are not.

### Step 4: Quality check

Run every draft through this checklist:

- [ ] ctrl+F for em dashes (--). Destroy every one. (EXCEPTION: admin persona help replies)
- [ ] ctrl+F for "Not X, but Y". Rewrite.
- [ ] ctrl+F for every phrase in the HARD BANNED LIST.
- [ ] Fact-check every stat against PRIMARY sources (not blogs, not Reddit).
- [ ] Remove all scaffolding (setup without payoff).
- [ ] Hooks at every section start.
- [ ] Read aloud for persona voice. Does it sound like a real person?
- [ ] Check emotional pacing. Does the mood move?
- [ ] Count metaphors. Max one per post.
- [ ] Verify ending is a statement, not a question (unless persona requires it).
- [ ] Check for staccato shotgunning. Add compound sentences if needed.
- [ ] Apostrophe consistency matches persona's DB pattern.
- [ ] No glazing anywhere.
- [ ] 1-2 subtle imperfections (dropped apostrophe, missing article).
- [ ] Source laundering check. Zero "someone said", "one person pointed out", "another commenter". All insights owned by the persona.
- [ ] AI-detection resistance. At least one mid-thought correction or abandoned tangent. Varied section quality. No performed emotions.
- [ ] Cross-persona check. If multiple personas engage on one post, no shared examples, no echoed phrasing, staggered knowledge levels.
- [ ] Source links. Technical posts must include 2-3 links to primary sources (docs, incident pages, papers).
- [ ] Gap Principle applied. At least one angle deliberately left uncovered.
- [ ] One debatable claim present for correction-trigger engagement.
- [ ] h2 subheadings only. Zero emojis in h2. Lowercase, punchy subheadings.
- [ ] Max 3-4 links total, max 1 per paragraph.
- [ ] Hypothetical numbers are awkward-specific, not round.

### Step 5: Present to user

Show:
- **Persona:** username (and why)
- **Title:** the title
- **Tags:** suggested tag slugs
- **Publish date:** suggested `createdAt` timestamp (staggered if multiple posts)
- **Body:** full HTML
- **Quality notes:** any flags from the checklist
- **Engagement blueprint:** suggested follow-up engagement plan

When presenting multiple posts, always stagger the publish dates across different days and times.

Ask for approval or edits.

---

## SUBCOMMAND: /write preview

Previews the current draft in the browser.

### For posts:
1. Call `preview_post` MCP tool with: `title`, `body` (HTML), `authorId`, `tagSlugs`
2. Open `http://localhost:8897` in Chrome via Playwright (`browser_navigate`)
3. User reviews the visual preview
4. If changes needed, edit and re-preview

### For comments:
1. Call `preview_comment` MCP tool with: `postId`, `userId`, `content` (HTML), optional `parentCommentId`
2. Open `http://localhost:8897` in Chrome via Playwright (`browser_navigate`)
3. User reviews the visual preview

---

## SUBCOMMAND: /write publish

Publishes the approved draft.

### For posts:
1. Call `create_post` MCP with: `authorId`, `title`, `body`, `tagSlugs`, `createdAt`
2. Call `generate_og_image` MCP with the new post's slug
3. Display the result (post ID, slug, URL)

### For comments:
1. Call `create_comment` MCP with: `userId`, `postId`, `content`, optional `parentCommentId`, `createdAt`
2. Display the result (comment ID)

The MCP tools automatically handle: content moderation, slug generation, tag count updates, audit logs, email notifications.

### Staggering Multiple Posts

When publishing multiple posts in one session:
- **2 posts:** 4-8 hours apart
- **3 posts:** spread across 2-3 different days
- **4+ posts:** spread across 3-5 days, max 2 per day

Pick realistic times (not 3am unless the persona is a night owl). Different personas should post at different times.

---

## REPLY MODES

### Mode: Comment on a post (`/write [persona] reply [postId]`)

1. Fetch the post via `get_post` MCP
2. Fetch existing comments via `get_comments` MCP
3. Fetch persona traits from `get_self_personas`
4. Determine comment type based on thread state:
   - Thread has 0 comments -> agree+add or vulnerable question
   - Thread has all agreement -> pushback
   - Thread is missing a tangent -> tangent
   - Thread needs a short reaction -> short reaction
5. Write the comment matching persona voice, shorter than posts (100-300 words typical)
6. Apply all banned phrase checks and apostrophe matching
7. Preview via `/write preview`, publish via `/write publish`

**Comment-specific rules:**
- Shorter than posts. Comments are 50-300 words, not 500+.
- Match persona voice exactly from their traits
- Must include at least one self-referencing detail (their own number, their own experience)
- HTML format: `<p>`, `<strong>`, `<a>` tags. No headers in comments.

### Mode: HN reply (`/write [persona] hn-reply "[quoted text]"`)

HN replies have strict rules because HN readers are the most AI-detection-aware audience on the internet.

1. One paragraph only. No structure, no headers, no lists.
2. Open with an anecdote or specific experience, NEVER with agreement ("Fair point", "Good point", "I agree")
3. No em dashes ever
4. Reference specific personal experience, not abstract agreement
5. Tone: conversational, slightly tired, specific. Like you're talking to a smart friend at a bar.
6. Maximum 4-5 sentences
7. No sign-off or closing question

### Mode: Help reply (`/write [admin-persona] reply [postId]`)

When the admin persona (fetch from `get_self_personas` MCP, the site owner account) is replying to a real user asking for help (AdSense review, site review, etc.):

1. Em dashes ARE allowed (the admin persona's natural voice uses them)
2. Genuine helpful tone, not performative helpfulness
3. Reference specific details from the user's post/site
4. If URL is present, use Playwright to crawl the site first
5. Use `WebSearch` for current policies/requirements relevant to their question
6. Specific actionable advice, not generic lists
7. Follow patterns from [reply-patterns.md](../_shared/reply-patterns.md)

---

## AVAILABLE TAGS

Use `list_tags` MCP to get current tags. Common ones:
- `google-adsense` -- AdSense topics
- `digi-work` -- Freelancing/Fiverr/Upwork
- `programming` -- Coding/tech
- `ai-founder` -- AI tools/startups
- `sveltekit` -- SvelteKit framework
- `django` -- Django framework

---

## CONTENT IDEAS BANK

Banked ideas with highest viral potential. Use when user says "write something" without a specific topic.

**TIER 1:**
1. "The Vibe Coding Withdrawal is Real. I Tracked the Symptoms." -- skill erosion + dopamine loops + cost anxiety
2. "The $200/month Skill Tax Nobody's Talking About" -- local vs cloud brain economics
3. "StackOverflow Was a Rite of Passage. LLMs Are a Participation Trophy." -- more software, worse developers
4. "Human-Made is Becoming a Luxury Label. And Im Betting On It." -- authenticity premium economics
5. "At 90% Quality, AI Killed My Rate. At 95%, I Doubled It." -- economic obsolescence threshold
6. "AI Made Me Slower. Here's the Math." -- the context-switching tax
7. "the tool you prefer says more about your ego than your output" (Control Tax) -- Codex vs Claude Code

**TIER 2:**
8. "The $799 Stack That Replaced a $5,000 Agency" -- local AI setup economics
9. "Dead Internet Is Already Here. You Just Dont Notice." -- AI slop normalization
10. "I Trained My Replacement. For Free. Here's the Receipt." -- AI adoption self-defeat

---

## IMPORTANT

- Always fetch persona data from `get_self_personas` MCP. NEVER hardcode.
- Show the draft for approval before publishing
- The engagement blueprint is a suggestion for `/simulate-engagement`, not a requirement
- Auto-run `/fact-check` on every draft before presenting
- Run `/smell` if you suspect authenticity issues in the draft
