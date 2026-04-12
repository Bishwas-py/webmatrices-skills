---
name: detect-fake-engagement
description: Scan Webmatrices posts and comments for patterns that look artificial, AI-generated, or suspiciously coordinated. Use when asked to audit engagement quality, check for fake-looking content, or verify organic appearance of discussions.
disable-model-invocation: true
argument-hint: [post-slug-or-scope]
---

# Detect Fake Engagement

Scan existing Webmatrices engagement for patterns that would look artificial to a real visitor. This skill applies engagement psychology research in reverse — instead of creating organic-looking engagement, it flags anything that fails the organic test.

For the psychology behind these patterns, see [engagement-psychology.md](../_shared/engagement-psychology.md).
For what real engagement looks like, see [reply-patterns.md](../_shared/reply-patterns.md).

## Arguments

- `[post-slug]` — audit a specific post and its comments
- `[scope: recent / all / persona-name]` — audit multiple posts or a specific persona's activity
- No args — scan recent posts and flag issues

---

## WHAT TO SCAN FOR

### Category 1: AI Voice Tells

These are the textual signals that make content feel AI-generated. Check every comment against this list.

| Signal | Severity | Example |
|--------|----------|---------|
| Em dash (—) in comments | HIGH | "the real problem — and nobody talks about this — is..." |
| "Not X, but Y" structures | HIGH | "its not about traffic, but about intent" |
| Bullet points in conversational comments | HIGH | Comments formatted as advice lists |
| Too helpful / too complete | MEDIUM | Comment covers every angle, leaves nothing to add |
| Too balanced ("on one hand... on the other") | MEDIUM | Hedging instead of taking a position |
| Perfect grammar throughout | MEDIUM | No contractions skipped, no casual errors |
| AI vocabulary | HIGH | "moreover", "crucial", "delve", "leverage", "comprehensive" |
| ChatGPT openers | HIGH | "Great question!", "I'd be happy to help!", "Absolutely!" |
| Structured responses in casual contexts | MEDIUM | Headers, numbered lists where a paragraph would be natural |
| No self-reference | MEDIUM | Comment gives advice without ever mentioning their own experience |
| Excessive enthusiasm / glazing | MEDIUM | "This is an AMAZING post!", "What a great insight!" |

### Category 2: Timing Anomalies

| Signal | Severity | What it looks like |
|--------|----------|--------------------|
| Instant comments | HIGH | Comment posted within 5 minutes of post |
| Evenly spaced comments | HIGH | Comments exactly 1 hour apart, or every 30 minutes |
| All comments same time of day | MEDIUM | Every comment posted between 2-3 PM |
| 3 AM activity | MEDIUM | Persona commenting outside their plausible timezone |
| All engagement on same day | HIGH | Post gets 5 comments all on day 1, then nothing ever |
| Burst without silence | MEDIUM | Constant steady engagement with no gaps |

### Category 3: Thread Dynamics

| Signal | Severity | What it looks like |
|--------|----------|--------------------|
| 100% agreement | HIGH | Every comment agrees. No pushback, no tangent, no questions |
| No thread drift | MEDIUM | Comments 5, 6, 7 are still exactly on-topic |
| All comments same length | MEDIUM | Every comment is 150-200 words. No short reactions, no long ones |
| No self-reference in any comment | MEDIUM | Nobody mentions their own experience, site, numbers |
| Perfect response rate | HIGH | Every single post has exactly 2-3 comments |
| OP always writes more than commenters | MEDIUM | Asymmetric effort in the wrong direction |

### Category 4: Persona Consistency

| Signal | Severity | What it looks like |
|--------|----------|--------------------|
| Voice bleed | HIGH | Two personas using the same verbal tics or sentence structures |
| Cross-persona vocabulary | HIGH | digitaldave01 using techwizardrino's "my brother in christ" |
| Identical opinion patterns | MEDIUM | All personas agree on everything. No personality conflicts. |
| Same persona everywhere | HIGH | One persona comments on every single post |
| Unrealistic activity | MEDIUM | Persona comments 8 times in one day |
| No silent days | MEDIUM | Persona is active every single day for weeks |

### Category 5: Content Patterns

| Signal | Severity | What it looks like |
|--------|----------|--------------------|
| Generic advice | HIGH | "1. Create quality content 2. Improve SEO 3. Be patient" |
| No specific numbers | MEDIUM | Vague claims without data. "Significant improvement" vs "$47.23" |
| No URLs or tools named | LOW | Real practitioners name their tools |
| Every comment ends with a question | MEDIUM | Formulaic engagement-driving |
| Mirror structure | HIGH | Multiple comments following the exact same template |
| No contrarian takes anywhere | MEDIUM | Community feels like an echo chamber |

---

## WORKFLOW

### Step 1: Gather data

Based on scope:

**Single post:** Use `list_posts` with `search` to find the post. Get all comments.

**Recent posts:** Use `list_posts` to get the last 20-30 posts with their comments.

**Persona audit:** Use `list_posts` filtered by author, then check all their comments across posts.

For each post, extract:
- Post title, author, creation timestamp
- All comments with: author, content, timestamp, parentCommentId
- Time gaps between consecutive comments
- Comment lengths (word count)

### Step 2: Run detection checks

For each post/thread, check against all 5 categories above. Score each finding:

- **HIGH severity:** would make a real visitor suspicious immediately
- **MEDIUM severity:** contributes to an overall "uncanny valley" feeling
- **LOW severity:** minor, but worth noting if combined with other signals

### Step 3: Generate the report

Present findings as a structured report:

```
ENGAGEMENT AUDIT REPORT
═══════════════════════

Scope: last 20 posts
Posts scanned: 20
Posts flagged: 7
Overall health: MODERATE RISK

───────────────────────

POST: "i made $47.23 from adsense last month"
Author: warmreboot | Posted: 2026-04-10 14:00 UTC
Comments: 3 | Flags: 2

  FLAG [HIGH]: All 3 comments agree. Zero pushback or tangent.
  → Recommendation: add a mild disagreement comment or let it be

  FLAG [MEDIUM]: Comments spaced exactly 3 hours apart (14:00, 17:00, 20:00)
  → Recommendation: future engagement should use irregular spacing

  PASSING: Comment lengths vary (87, 210, 45 words) ✓
  PASSING: Thread has self-referencing data in 2/3 comments ✓
  PASSING: No AI vocabulary detected ✓

───────────────────────

POST: "backlinks are dead for sites under 1000 pages"
Author: serpsherpa | Posted: 2026-04-08 09:00 UTC
Comments: 4 | Flags: 4

  FLAG [HIGH]: Comment by digitaldave01 contains em dash
  → Text: "the real problem — at least for my sites — is..."
  → Fix: edit comment to remove em dashes

  FLAG [HIGH]: Comment by techwizardrino uses "Not X, but Y"
  → Text: "its not about the backlinks, but about the authority signals"
  → Fix: rewrite to remove this structure

  FLAG [MEDIUM]: No thread drift. All 4 comments stay exactly on backlinks.
  → Recommendation: one comment should have gone tangential

  FLAG [MEDIUM]: romanking commented at 03:15 UTC (3 AM for his pattern)
  → Recommendation: timestamp should be during active hours

───────────────────────

PERSONA HEALTH CHECK

  techwizardrino: 6 comments this week, active 5/7 days ✓
  romanking: 8 comments this week ⚠️ (high for one persona)
  digitaldave01: 2 comments this week ✓
  serpsherpa: 4 comments this week, active 4/7 days ✓
  warmreboot: 5 comments this week ✓
  bishwasbhn: 1 comment this week ✓

  ⚠️ romanking is overexposed. Consider 2-3 silent days.

───────────────────────

GLOBAL PATTERNS

  Agreement rate across all threads: 85% ⚠️
  → Target: 70%. Need more pushback comments.

  Average time to first comment: 22 minutes ⚠️
  → Target: 30-90 minutes. Comments are arriving too fast.

  Posts with 0 engagement: 2/20 ✓
  → Some posts "failing" is realistic.

  Unique commenter diversity: 5 personas across 20 posts ✓
```

### Step 4: Prioritize fixes

After the report, provide a prioritized action list:

```
PRIORITY FIXES (do these first):
1. Edit comment on "backlinks are dead" to remove em dash
2. Edit comment on "backlinks are dead" to remove "not X but Y"
3. Add a pushback comment to "i made $47.23" thread
4. Give romanking 3 silent days this week

WATCH LIST (monitor but not urgent):
- Agreement rate trending high (85%). Next simulate-engagement session should skew toward pushback.
- First-comment timing is too fast. Increase minimum gap to 30 min.
```

---

## WHEN TO RUN THIS SKILL

### Proactive (recommended cadence)
- **Weekly:** quick scan of last 7 days of engagement
- **After bulk simulate-engagement sessions:** immediately audit what was just created
- **Before inviting real users:** full audit to catch anything suspicious

### Reactive
- When a real user comments something like "this feels like bots" or "are these real people"
- When engagement metrics suddenly change
- When adding a new persona (check their voice isnt bleeding from existing ones)

---

## FIXING DETECTED ISSUES

### For AI voice tells
- Edit the comment directly using MCP tools if possible
- If the comment cant be edited, note it for future avoidance
- Add the specific phrase to your awareness for future simulate-engagement sessions

### For timing anomalies
- Cannot fix past timestamps retroactively (unless MCP supports editing createdAt)
- Note the pattern and adjust timing rules in future sessions
- Consider adding a comment with a more realistic timestamp to break the pattern

### For thread dynamics
- Add a missing comment type (pushback, tangent, short reaction)
- Use simulate-engagement to add the needed diversity

### For persona consistency
- Identify which persona is overexposed and schedule silent days
- Check recent comments for voice bleed and rewrite if needed
- Review persona profiles in viral-writer to ensure distinctness
