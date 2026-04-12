---
name: simulate-engagement
description: Orchestrate organic-looking engagement on Webmatrices posts. Scans posts, picks personas, decides comment types with realistic timing gaps, and outputs an engagement plan for approval before executing. Use when asked to simulate engagement, seed comments, reply to posts, or create organic-looking discussion.
disable-model-invocation: true
argument-hint: [post-slug-or-scope]
---

# Simulate Engagement

Orchestrate organic-looking engagement across Webmatrices posts. This skill replaces manual one-off replies with a strategic engagement system that considers personas, timing, comment types, thread dynamics, and platform phase.

For engagement psychology research, see [engagement-psychology.md](../_shared/engagement-psychology.md).
For reply quality patterns, see [reply-patterns.md](../_shared/reply-patterns.md).
For content patterns, see [content-engine.md](../_shared/content-engine.md).

## Arguments

- `[post-slug]` — simulate engagement on a specific post
- `[scope: recent / unanswered / all]` — scan posts matching a scope
- No args — scan recent posts and suggest which need engagement

---

## CORE PRINCIPLES

### The 70/20/10 Rule

Every seeded thread follows this distribution:

- **70%** of comments agree with the post but add new information, personal experience, or data
- **20%** mildly disagree, ask challenging questions, or offer an alternative perspective
- **10%** are tangential — loosely related, going off-topic the way real threads do

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

Each persona has implied timezone behavior. Dont have them commenting at 3 AM their time.

| Persona | Active hours (UTC) | Pattern |
|---------|-------------------|---------|
| techwizardrino | 06:00-22:00 | Late-night researcher energy, active evenings |
| serpsherpa | 08:00-18:00 | Business hours, rarely weekends |
| digitaldave01 | 07:00-20:00 | Early riser, fades by evening |
| bishwasbhn | 05:00-21:00 | Nepal time, erratic schedule |
| romanking | 10:00-23:00 | Late starter, night owl |
| warmreboot | 06:00-19:00 | Anxious early-morning checker |

### Use `createdAt` parameter

All comments should use the `createdAt` parameter on `create_comment` to set the appropriate timestamp. Never let all comments land at "right now."

---

## COMMENT TYPES

### Type 1: Agreement + Addition (70% of comments)

The commenter agrees but adds their own experience, data, or angle. This is the most common real comment type.

**Structure:**
```
[Brief agreement or recognition]
[Their own experience/data that extends the post]
[Optional question or observation]
```

**Example (warmreboot on an AdSense post):**
> seeing the exact same thing on my end. rpm went from $4.80 to $1.90 basically overnight and i havent changed anything on the site. 23 posts, all original, decent session duration. the part that bugs me is my search console shows MORE impressions than last month but the revenue is tanking. makes no sense unless theyre devaluing certain niches entirely. are you seeing this across all your sites or just specific ones?

**Rules:**
- Must include at least one specific personal detail (number, date, tool name)
- Must extend the conversation, not just agree
- Tone matches the persona voice exactly
- 80-250 words typically

### Type 2: Mild Pushback (20% of comments)

The commenter disagrees partially or offers an alternative perspective. NOT hostile. Constructive disagreement.

**Structure:**
```
[Acknowledge the post has a point]
[But here's where they differ, with their own evidence]
[Reframe or alternative interpretation]
```

**Example (romanking pushing back on a "backlinks are dead" post):**
> i get where youre coming from but this feels like survivorship bias. youre looking at sites that ranked without backlinks and concluding they dont matter. thats like looking at lottery winners and concluding everyone should play. my niche sites with 0 backlinks are stuck at page 3. the ones where i actually did outreach are on page 1. the difference isnt subtle. maybe backlinks are dead for YOUR niche but calling it dead universally is a stretch.

**Rules:**
- Never hostile or dismissive
- Must have their own evidence, not just "I disagree"
- Should feel like a real person who respects OP but has different experience
- Generates the most reply-chain potential

### Type 3: Tangent (10% of comments)

The commenter takes the conversation in an adjacent direction. This is what makes threads feel real — the natural drift.

**Structure:**
```
[Loosely connect to the post]
[Go off in their own direction]
[May or may not come back to the original topic]
```

**Example (digitaldave01 going tangent on an AI tools post):**
> this is making me think about something slightly different. ive been using these tools to automate my client reporting and honestly the part that keeps me up at night isnt whether the output is good enough. its whether my clients will figure out im using AI and feel like theyre overpaying. like is there an ethical line here? im saving 6 hours a week on reports but charging the same rate. does anyone else think about this or am i overthinking it

**Rules:**
- Must feel like a natural association, not random
- Often reveals something personal about the commenter
- Can introduce a new sub-topic the thread can explore
- Usually includes a question that invites further tangent

### Type 4: Short Reaction (sprinkle in)

Not every comment is a paragraph. Real threads have short reactions too:
- "this." or "yep same"
- "lol the control tax. stealing that"
- "saving this thread"
- "man this hit different"

Use 1-2 per thread maximum. Usually from personas who are "lurkers" on this topic.

---

## REPLYING TO REAL USERS (PHASE 2-3)

When the platform has real users posting, engagement shifts:

### Priority: Real user posts get engagement first
- Always prioritize replying to real user posts over persona-to-persona engagement
- Real users who get no replies will leave. This is the #1 retention risk.

### How to reply to real users

1. **Research their specific situation** — if they shared a URL, use Playwright to crawl it. If they mentioned numbers, reference them specifically.
2. **Never be generic** — see [reply-patterns.md](../_shared/reply-patterns.md). "I actually looked at YOUR thing" is the #1 engagement pattern.
3. **Pick the right persona** — match topic expertise, not just availability.
4. **One persona replies first, others can follow** — but only if they have something genuinely different to add. Dont have 3 personas all saying "great post."

| Real user post type | Best persona | Reply approach |
|--------------------|--------------|----|
| AdSense help (URL shared) | bishwasbhn | Crawl site with Playwright, give specific audit |
| AdSense help (no URL) | warmreboot | Commiserate with own experience, ask for their numbers |
| Fiverr help | techwizardrino | Buyer perspective, practical tips |
| SEO question | serpsherpa or digitaldave01 | Storytelling or data-driven |
| Hot take / opinion | romanking | Sharp pushback or agreement |
| Beginner question | digitaldave01 | Welcoming, share own learning journey |

### Research tools for replies

#### When URL is present
Use Playwright to crawl the user's site/gig. Extract specific details to reference.

#### When no URL
Use WebSearch for current policies, algorithm changes, or best practices relevant to their question.

#### Reddit context
Search Reddit for what people are currently saying about the same topic. Use `search_reddit` with the post's core topic, targeting relevant subreddits from [content-engine.md](../_shared/content-engine.md).

---

## WORKFLOW

### Step 1: Scan the platform

Use `list_posts` to get recent posts. For each post, note:
- Post title, author, age, tag
- Number of existing comments
- Whether author is a real user or persona
- Whether existing comments cover diverse perspectives

### Step 2: Decide what needs engagement

Apply these filters:

**Must engage (high priority):**
- Real user posts with 0 comments (retention risk)
- Real user posts with only 1 generic comment
- Posts with engagement blueprints from viral-writer

**Should engage (medium priority):**
- Persona posts that look dead (0 comments after 24hrs)
- Posts where all existing comments agree (needs pushback)
- Posts on trending topics

**Skip (low priority):**
- Posts already with 3+ diverse comments
- Very old posts (7+ days) unless theres a reason
- Some posts deliberately (not everything gets engagement)

### Step 3: Build the engagement plan

For each post that needs engagement, determine:
1. **Which personas** comment (check who hasnt been active recently)
2. **What type** of comment (apply 70/20/10)
3. **What timestamp** (apply all timing rules)
4. **Comment content summary** (1-2 sentence description of what each comment will say)

Check if viral-writer left an engagement blueprint for this post. Use it as a starting point.

### Step 4: Present the engagement plan

Show the user a table:

```
ENGAGEMENT PLAN
═══════════════

Post: "i made $47.23 from adsense last month and honestly im not sure its worth it"
Author: warmreboot | Age: 6 hours | Comments: 0

  Comment 1: techwizardrino (agree+add)
  Time: ~45 min after post (createdAt: 2026-04-12T15:30:00Z)
  Summary: shares his own adsense numbers ($92/month), says the real question
           is opportunity cost vs the learning value of maintaining a site

  Comment 2: romanking (mild pushback)
  Time: ~4 hours after post (createdAt: 2026-04-12T18:45:00Z)
  Summary: pushes back — $47 is $47, most side projects make $0.
           the real problem is comparing to influencer income numbers

  Comment 3: digitaldave01 (tangent)
  Time: next day morning (createdAt: 2026-04-13T08:15:00Z)
  Summary: goes off on whether adsense is even the right monetization
           for small sites, wonders about affiliate vs display ads

───────────────

Post: "backlinks are dead for sites under 1000 pages"
Author: serpsherpa | Age: 2 days | Comments: 1 (agreement only)

  Comment 1: romanking (pushback)
  Time: ~2 hours from now (createdAt: 2026-04-12T16:00:00Z)
  Summary: disagrees with specific evidence from his own sites

───────────────
Distribution check: 3 agree, 2 pushback, 1 tangent (50/33/17) ✓ close to 70/20/10
Persona activity: techwizardrino (1 today), romanking (2 today), digitaldave01 (1 tomorrow) ✓
Timing gaps: all gaps > 2 hours ✓
```

Ask for approval or modifications.

### Step 5: Write the comments

On approval, write each comment following:

1. **Persona voice** — match exactly from viral-writer persona profiles
2. **Authenticity markers** — apply apostrophe psychology, imperfection principle, rhythm from viral-writer
3. **Banned list** — check against viral-writer's hard banned list. No em dashes, no "not X but Y", no AI vocabulary.
4. **Length variation** — not every comment should be 200 words. Mix: one longer (200-400), one medium (100-200), one short (50-100).
5. **HTML format** — `<p>`, `<strong>`, `<ol>`, `<ul>`, `<code>`, `<a>` tags

Show each draft for approval.

### Step 6: Execute via MCP

On approval, use `create_comment` for each:
- `userId`: the persona's user ID
- `postId`: the target post ID
- `content`: the HTML comment
- `parentCommentId`: if replying to a specific comment in the thread
- `createdAt`: the planned timestamp from the engagement plan

The MCP `create_comment` tool automatically:
- Runs content moderation (`guardContent`)
- Updates post comment count and last activity
- Sends email notification to the post/comment author
- Creates audit log

### Step 7: Verify

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
- Create question posts FROM personas to model the behavior you want real users to adopt.

### Phase 2: SEEDED (50-200 posts, some real users)
- Shift to engaging WITH real users. Reply to every real user post.
- Persona-to-persona engagement drops to 40-50%.
- Goal: make real users feel heard and valued.

### Phase 3: TRACTION (200+ posts, real users commenting)
- Personas step BACK. 20-30% of engagement is seeded.
- Let gaps exist for real users to fill.
- Only seed when threads need diversity of opinion.

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
- [ ] No persona uses another persona's verbal tics
- [ ] Time-of-day is realistic for each persona
- [ ] Not every post in the scope got engagement (some should be skipped)
- [ ] All comments pass viral-writer's banned list check
- [ ] `createdAt` timestamps are set, not defaulting to "now"
