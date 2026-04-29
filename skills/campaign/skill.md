---
name: campaign
description: Email campaign workflow with safety states. Drafts, previews, and sends targeted emails to Webmatrices audience segments. Three states DRAFT/PREVIEWED/SENT ensure you never accidentally blast 27K users. Backed by 30-day Brevo performance data — uses proven patterns, blocks the proven losers. Use when asked to send an email campaign, promote a post, email users, or create a newsletter.
disable-model-invocation: true
argument-hint: ["recent" / postId/slug / "description of what to promote"]
---

# Campaign

Email campaign workflow with safety states. Builds, previews, and sends targeted email campaigns to Webmatrices audience segments. Three-state machine ensures you never accidentally send to 27K users. **Performance rules in this skill are derived from real Brevo data (April 2026 analysis, internal opens filtered).**

---

## STATE MACHINE

```
DRAFT → PREVIEWED → SENT
```

| State | What happened | What can happen next |
|-------|-------------|---------------------|
| DRAFT | Email content built, segments chosen | /campaign preview (sends to the site owner only) |
| PREVIEWED | The site owner received the test email | /campaign send (fires to full audience) |
| SENT | Email delivered to segments | Done |

**Rules:**
- `/campaign [anything]` = always starts in DRAFT state. Never sends.
- `/campaign preview` = sends to the site owner ONLY (fetch admin user from `get_self_personas` MCP). Moves to PREVIEWED.
- `/campaign send` = fires to full segments. If NOT previewed first, asks "Are you sure you want to send without previewing? This will go to [N] users."
- You can re-draft at any point. Going back to DRAFT resets the state.

---

## WHAT THE DATA SHOWS — WORKS, KINDA WORKS, DOESN'T WORK

All numbers below come from Brevo events for the trailing 30 days, internal users (`is_internal=true` plus `bishwasbh+*` aliases) filtered out. **Use these as the empirical baseline. Do not draft a campaign that ignores them.**

### TIER 1 — CHAMPIONS (run these on rotation)

| Pattern | Example | Sent | Open % | CTR % |
|---------|---------|-----:|-------:|------:|
| Personalized lapsed-user story | "5 sites got reviewed this week, [Name]" | 49 | **51.0%** | **18.4%** |
| Personalized "it took X before Y" hook | "it took 40 posts before google sent a single visitor, [Name]" | 48 | **54.2%** | **10.4%** |
| Comment notifications on hot threads | "New comment on your post 'getting rejected by AdSense...'" | 6 | 50.0% | 50.0% |
| Reply notifications | "New reply to your comment on '...'" | 8 | 37.5% | 25.0% |

**What unites the champions:**
1. First-name personalization in the subject (`, [Name]` suffix)
2. Concrete number anchored in the hook (5 sites, 40 posts)
3. First-person story framing ("it took X before Y") — not announcement framing
4. Sent in batches of 50–200, never as mass blasts
5. Topic recipient genuinely cares about (AdSense rejection, real engagement on their post)

**Run these templates weekly against fresh segments.** They are 10–15× more effective than any broadcast.

### TIER 2 — KINDA WORKS (good open, weak click — fix the body)

| Subject | Sent | Open % | CTR % | What's wrong |
|---------|-----:|-------:|------:|--------------|
| "This week on Webmatrices: Vercel breach, vibe coding..." | 84 | 27.4% | **0%** | Body buries CTA |
| "Your old content might be killing your new rankings" | 80 | 21.2% | **0%** | Story told in-email; no reason to click |
| "A Roblox cheat just took down Vercel..." | 57 | 21.1% | **0%** | Multiple links compete |
| "your traffic is down but your rankings are fine" | 117 | 20.5% | 3.4% | Working but underperforming the open rate |
| "a developer let ai write all his code for 4 months" | 141 | 17.7% | 1.4% | Same — opens are great, clicks lag |

**Diagnosis:** news-event hooks pull opens but the email bodies leak readers. The reader opens, doesn't see one clear destination, bounces. Fix: single button, single thread, body is a tease not a recap. See "Body design" below.

**Solid workhorses (no fix needed, run as-is):**

| Subject | Sent | Open % | CTR % |
|---------|-----:|-------:|------:|
| "Quick fix — here's the correct link, [Name]" | 51 | 23.5% | 3.9% |
| "Your checklist is waiting, [Name]" | 51 | 23.5% | 5.9% |
| "you joined at the right time — 21 app ideas ready to build" | 234 | 10.7% | 1.3% |

### TIER 3 — DOESN'T WORK (do not draft these)

| Subject | Sent | Open % | CTR % | Why dead |
|---------|-----:|-------:|------:|----------|
| Mass blast: "what ai, side hustles, and remote work have in common" | **11,409** | **3.6%** | **0.25%** | Too large; stale list; segment fatigue |
| "We built something new, [Name]" | 116 | 8.6% | 0.9% | Pure announcement frame |
| "this week on webmatrices — 4 new app ideas + fresh posts" | 89 | 6.7% | **0%** | Recap framing |
| "4 new app ideas just landed on webmatrices" | 53 | 5.7% | **0%** | Recap framing |

**Banned subject patterns:**
- "[X] new posts/ideas/features just landed"
- "This week on Webmatrices" as opener (the actual format, not just words "this week" elsewhere)
- "We built something new"
- "[Number] [things] you might have missed"
- Any subject that telegraphs "this is a promotional update" before the recipient even opens

### Aggregate channel state

| Cohort | Open rate | CTR |
|---|---:|---:|
| Last 30 days, all sends | 5.61% | 0.81% |
| Excluding the one mass blast | **20.79%** | **5.11%** |
| Mass blast in isolation | 3.60% | 0.25% |

**Read:** the channel is healthy when used surgically. One bad send dragged the average from 21% to 6%. Repeat that mistake monthly and the platform's deliverability degrades for every future send.

---

## VOLUME & LIST HYGIENE RULES (NON-NEGOTIABLE)

The MCP enforces a **hard cap of 500 recipients per send**. There is no override. To reach more users, batch multiple campaigns and chain `excludeUserIds` so the second campaign excludes everyone the first reached.

These rules exist because the 2026-04 mass blast (sent before the hard cap was tightened) hit 2.93% hard bounce rate — above the ESP threshold of 2%, throttling above 5%.

### Volume strategy

| Goal recipient count | How to do it |
|---|---|
| ≤ 500 | Single `send_targeted_email` call. Sweet spot. |
| 500–1,500 | 2–3 sequential calls, each ≤ 500, chained with `excludeUserIds` so no overlap |
| 1,500+ | Reconsider. Splitting a topic across that many users almost always lowers per-send performance. Narrow the segment instead. |
| `all_active` (~27K) | Blocked by MCP. Only allowed for critical TOS/policy updates after explicit override. |

### What the MCP does automatically (don't duplicate this work)

- Excludes `is_internal=true` users — no skill-level filtering needed
- Excludes users with `emailNotifications=false`
- Verifies internal links (`/post/[slug]`) exist; blocks send on broken links
- Personalizes `{{username}}` and `{{firstName}}` placeholders
- Hard-caps at 500 recipients (501+ is blocked)
- Returns the list of `sentUserIds` so you can `excludeUserIds` in the next batch

### What the skill must add

- **Hard-bouncer exclusion.** The MCP doesn't track hard bounces (yet). After any send, manually note who bounced via Brevo dashboard and add their IDs to `excludeUserIds` for future sends.
- **12-month-silent exclusion.** Use `inactive_users` segment with `days: 365` to surface them, route to re-permission flow rather than including in promo campaigns.

### List rotation

Do not email the same segment more than once per 7 days unless it's a transactional event (comment notification, reply, account confirmation). Repeated hits cause segment fatigue.

---

## SUBJECT LINE PATTERNS (use these formulas)

### Formula A — Personalized story hook (CHAMPION)

```
[Concrete number] [outcome verb] [time/scope], [First Name]
```

Examples that work:
- "5 sites got reviewed this week, [Name]"
- "it took 40 posts before google sent a single visitor, [Name]"
- "$143 in adsense bills last month, [Name]"

Why it works: name signals 1:1 intent, number signals real story, lowercase signals personal.

### Formula B — News-event hook (Tier 2 — works for opens)

```
[Specific event/entity] [verb] [outcome]. [Optional context.]
```

Examples that work for opens (need body fix for clicks):
- "Google just confirmed GEO is real. They put it in the ads team."
- "A Roblox cheat just took down Vercel."
- "Your old content might be killing your new rankings."

Use only when there's a real news hook AND the body is single-thread, single-button.

### Formula C — Pain-point question (Tier 2)

```
[State of affairs] but [contradicting outcome] — [framing question]
```

Examples that work:
- "your traffic is down but your rankings are fine — and nobody can explain why"
- "you joined at the right time — 21 app ideas ready to build"

### Banned subject formulas

- ❌ "[Number] new [things] just landed on Webmatrices"
- ❌ "This week on Webmatrices: [recap]"
- ❌ "We built something new, [Name]"
- ❌ "[X] you might have missed"
- ❌ All-caps anywhere
- ❌ Excessive punctuation (multiple exclamation marks, ellipses)
- ❌ Emojis at the start of the subject (mid/end is fine in moderation)

---

## BODY DESIGN RULES

The skill provides **inner body HTML only** — paragraphs, optional inline links, optional `<strong>` for emphasis. The MCP wraps it with the email template (header, footer, unsubscribe) and appends the CTA button when you pass `ctaText` + `ctaUrl`. **Do not write the button HTML yourself. Do not write `<table>` wrappers. Do not write the email template chrome.**

The Tier 2 problem (high open, zero CTR) is bodies leaking readers. These rules exist to fix that.

### The single-destination rule

Every email has **one** destination — passed as `ctaUrl`. Do not put multiple inline links inside the body. If you absolutely must mention a secondary item, put it in a small `<p>` "P.S." line at the bottom with a single secondary inline link. The CTA button (rendered by the MCP from `ctaText`/`ctaUrl`) must be the dominant action.

### Body structure (inner HTML only)

What the skill writes:

```html
<p>hey {{firstName}},</p>

<p>[2-3 sentence personal frame — what made you write this email today.
Real and specific, first person, not corporate.]</p>

<p>[1 sentence specific tease about the thread — a number, a name,
a turn of phrase from the post. Not a summary. Leave the reader curious.]</p>

<p>[1 sentence why this matters to the recipient specifically — segment relevant.]</p>

<p>— bishwas</p>
```

The MCP appends the CTA button below this body using `ctaText` and `ctaUrl` parameters. The button styling, white-text span fix, and centering are all handled by the MCP. **Don't write button markup in the body.**

### Body length

- Total stripped text: **80–180 words**. Below 80 reads as spam. Above 180 the reader drops off before the button.
- The body is a **tease**, not a summary. If the email tells the whole story, there's no reason to click.

### Personalization

Use `{{firstName}}` and `{{username}}` in body and subject. The MCP substitutes them per-recipient. Fall-back behavior: if `firstName` is null, the MCP uses `username`.

### CTA parameters (passed alongside body)

- `ctaText`: action-oriented, max 5 words. ("Read the thread →", "See what changed →", "Drop your URL below →"). Never generic ("Click here", "Learn more").
- `ctaUrl`: relative path (e.g., `/post/your-slug`) or absolute. The MCP verifies the slug exists before sending.

### Personal tone markers

- First person ("I"), not corporate "we"
- Lowercase greeting ("hey {{firstName}}")
- One specific detail that signals real authorship (a number, a date, a personal admission like "i sat at my desk for a while after that")
- Sign-off with first name only

---

## INPUT

| Input | What it does |
|-------|-------------|
| `"recent"` | Finds last 2-3 published posts, builds campaign around them |
| Post ID | Builds campaign promoting that specific post |
| Post slug | Fetches post via `get_post_by_slug`, builds campaign around it |
| Description text | Builds campaign based on described theme/promotion |

---

## WORKFLOW

### Step 1: Gather content

**If "recent":**
1. Call `list_posts` to get the 5 most recent posts
2. Pick **one** standout post (highest comment count, freshest news angle, strongest hook)
3. Verify the slug exists

**If postId/slug:**
1. Fetch the post via `get_post` or `get_post_by_slug` MCP
2. Verify it exists

**If description:**
1. Determine the theme
2. Search for relevant posts using `search_posts_by_content` or `list_posts`
3. Pick **one** post that best matches

**CRITICAL: Single destination only.** The data is unambiguous — multi-link emails get 0% CTR even with 21% opens. Pick one thread.

### Step 2: Choose segments

Use `preview_audience` MCP to check segment sizes before choosing.

**Segment selection rules:**

1. Match segment to topic. AdSense post → AdSense segment, never "all users."
2. Default segment size target: **200–800 recipients**. The Brevo data shows performance peaks in this range.
3. Cap any single send at 1,500 unless explicitly justified.

**Available segments (real MCP names):**

| Segment | What it targets | Best for |
|---------|----------------|----------|
| `commenters` | Users who have commented at least once | High-engagement campaigns. Most reactive segment. |
| `post_authors` | Users who have published posts | Engagement boosts on related new posts |
| `lapsed_contributors` | Posted/commented but inactive `days+` | Reactivation of warm users (set `days: 14-30`) |
| `inactive_users` | Inactive for `days+` (default 30) | Re-engagement (set `days: 30/60/90/365`) |
| `new_users` | Joined in last `days` (default 7) | Welcome / starter-content campaigns |
| `checklist_users` | Created project checklists | Tool/workflow related campaigns |
| `premium_users` | Has `isPremium=true` | Premium-only announcements |
| `all_active` | Everyone active | **Blocked by MCP.** Critical TOS only. |

**Segment chaining:** when running multiple sends to overlapping segments, the MCP returns `sentUserIds` after each send. Pass that array as `excludeUserIds` on the next send so no user gets duplicates.

**Time-window parameter (`days`):** for `inactive_users`, `lapsed_contributors`, and `new_users`, the `days` argument tunes the window. Use it to surface the right cohort (e.g., `lapsed_contributors` with `days: 30` = posted/commented but quiet for a month).

### Step 3: Internal user exclusion (handled by MCP)

The MCP automatically excludes `is_internal=true` users from every `send_targeted_email` call. **You do not need to fetch persona IDs and add them to `excludeUserIds`.** This was previously a skill-level requirement; it's now baked into the MCP. Just use `excludeUserIds` for chaining across multiple sends and known hard-bouncers.

### Step 4: Pre-flight quality check

Before showing the draft, run this internal check. Score 1 point per ✅. **Block any draft scoring below 6/8.**

| Check | Pass criteria |
|-------|---------------|
| Subject matches a Tier 1 or 2 formula | First-name + number + story OR clear news-event hook |
| Subject is NOT a banned format | No "X just landed", no "this week on", no "we built" |
| Subject is under 70 characters | Mobile preview cuts off |
| Body provides single `ctaUrl` destination | One thread, one button |
| Body word count 80–180 (stripped) | Sweet spot for opens-to-click conversion |
| Body does NOT contain manual button HTML | MCP renders the button from `ctaText`/`ctaUrl` |
| Body does NOT contain template chrome | No `<table>` wrappers, no header/footer markup |
| Segment matches topic and ≤ 500 recipients | MCP hard-caps at 500 |

### Step 5: Present the DRAFT

Show:
- Subject line (with character count)
- Target segment(s) with estimated user count
- Excluded segments/users count
- Full HTML body
- Pre-flight score (e.g., "9/9 — ready to preview")
- STATE: DRAFT

Ask: "Ready to preview? Run `/campaign preview` to send a test to the site owner."

**NEVER auto-send. NEVER proceed past DRAFT without user action.**

---

## SUBCOMMAND: /campaign preview

1. Send the email to the site owner ONLY using `send_email` MCP with:
   - `userId: 1` (the site owner's user ID)
   - Full subject and body as built in the draft
   - Do NOT use `send_targeted_email` for preview — it picks from the segment, not the site owner
2. Move state to PREVIEWED
3. Tell the user: "Test email sent to the site owner. Check your inbox. When ready: `/campaign send`"

---

## SUBCOMMAND: /campaign send

1. Check state is PREVIEWED
   - If YES: proceed
   - If NO: warn "This campaign hasn't been previewed. Are you sure you want to send to [N] users without testing? Say 'yes send it' to confirm."
2. Re-run pre-flight checks (in case anything changed since draft)
3. Send via `send_targeted_email` MCP with all exclusions applied
4. Move state to SENT
5. Report: "Campaign sent to [N] users across [segments]. Subject: [subject]. Track results in Brevo dashboard after 48 hours."

---

## CAMPAIGN TYPES & WHEN TO USE EACH

### Reactivation (CHAMPION — use weekly)

Target: lapsed users (no login 30–90 days, opened at least one email in last 12 months).
Format: Tier 1 personalized story hook.
Subject: "5 sites got reviewed this week, [Name]" / "it took 40 posts before google sent a single visitor, [Name]" / variant.
Send size: 100–300 per batch.
Cadence: weekly.

### Hot-thread reply boost

Target: users who commented on related threads in last 30 days.
Format: Tier 2 news-event hook OR Formula C pain-point.
Subject: news angle from the post.
Send size: 80–250.
Cadence: 1× per major post.

### Onboarding (transactional)

Target: just-registered users.
Format: short, single CTA to a starter thread.
Subject: "Confirm your email" / "Welcome to Webmatrices!" — these already perform well (38–55% open).

### Re-permission (long-silent users)

Target: 12+ months no opens.
Format: explicit "still want to hear from us?" with single yes/no link.
Send size: 500 per batch max.
Cadence: once per stale segment, then drop them if no reply.

### NOT a campaign type

- ❌ Generic platform recaps ("this week on Webmatrices")
- ❌ Feature announcements ("we built something new")
- ❌ Mass blasts to all 27K users for any reason except a critical TOS/policy update

---

## SAFETY CHECKLIST (before any send)

- [ ] Post slug in `ctaUrl` is verified to exist (the MCP will block on broken slug — but check yourself first)
- [ ] Subject matches a Tier 1 or Tier 2 formula
- [ ] Subject is NOT a banned format
- [ ] Body is inner HTML only (paragraphs, no template chrome, no manual button markup)
- [ ] Body word count 80–180 (stripped)
- [ ] Single `ctaUrl` destination
- [ ] Send size ≤ 500 (MCP hard cap)
- [ ] Segment matched to topic (not `all_active`)
- [ ] Hard-bouncers added to `excludeUserIds` (manual until MCP supports it)
- [ ] Multi-batch sends chain `excludeUserIds` from previous `sentUserIds`
- [ ] Pre-flight score ≥ 6/8
- [ ] Preview sent to site owner before full send (or explicit override with reason)

---

## QUARTERLY REVIEW

The performance numbers in this skill are from the trailing 30 days as of 2026-04-27. Re-run the Brevo analysis quarterly:

1. Pull `/v3/smtp/statistics/events` for last 90 days
2. Filter out `is_internal=true` users (and `bishwasbh+*` aliases)
3. Bucket by template (strip `, [Name]` suffix)
4. Update the TIER 1 / TIER 2 / TIER 3 tables in this skill if patterns shift
5. Update the Wiki diagnosis at `wiki/Forum-Engagement-Diagnosis-YYYY-MM.md`

If a pattern stops working, retire it from TIER 1 immediately. The empirical data leads the rules.

---

## IMPORTANT

- **State machine is non-negotiable.** DRAFT → PREVIEWED → SENT.
- **The empirical performance tiers lead the rules.** Do not draft against patterns proven dead.
- **Single destination per email.** One `ctaUrl`. Multi-link emails get 0% CTR even at 21% opens — proven repeatedly in the data.
- **MCP hard-caps at 500 recipients per send.** Larger reach = multiple chained sends.
- **Body is inner HTML only.** Paragraphs and inline tags. The MCP wraps the template, adds the button, handles white-text fix, personalizes, and excludes internal/opted-out users automatically.
- **Personal tone, not corporate.** First person, lowercase greeting, one specific detail that signals real authorship.
- **Quarterly performance review.** Update tiers when data shifts. Reference `wiki/Forum-Engagement-Diagnosis-YYYY-MM.md`.
