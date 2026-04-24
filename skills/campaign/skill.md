---
name: campaign
description: Email campaign workflow with safety states. Drafts, previews, and sends targeted emails to Webmatrices audience segments. Three states DRAFT/PREVIEWED/SENT ensure you never accidentally blast 27K users. Use when asked to send an email campaign, promote a post, email users, or create a newsletter.
disable-model-invocation: true
argument-hint: ["recent" / postId/slug / "description of what to promote"]
---

# Campaign

Email campaign workflow with safety states. Builds, previews, and sends targeted email campaigns to Webmatrices audience segments. Three-state machine ensures you never accidentally send to 27K users.

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
2. Pick the 2-3 most engaging (highest comment count, most interesting titles)
3. Verify each post exists and has a valid slug

**If postId/slug:**
1. Fetch the post via `get_post` or `get_post_by_slug` MCP
2. Verify it exists
3. Optionally include 1-2 related recent posts for the email body

**If description:**
1. Determine the theme
2. Search for relevant posts using `search_posts_by_content` or `list_posts`
3. Pick 2-3 posts that match the theme

**CRITICAL: Verify every post slug exists before building the email.** Broken links in emails destroy trust.

### Step 2: Choose segments

Use `preview_audience` MCP to check segment sizes before choosing.

**Common segments:**

| Segment | Description | Best for |
|---------|------------|---------|
| All users | Everyone | Major announcements only |
| AdSense | Users interested in AdSense | AdSense content |
| SEO | SEO-focused users | SEO content |
| Web Dev | Developer-focused users | Technical content |
| Active commenters | Users who commented recently | Engagement campaigns |

**Segment chaining with excludeUserIds:**

When targeting multiple segments, chain them to avoid duplicate emails:

```
Segment 1: adsense users → sends to [N] users, collect their IDs
Segment 2: seo users, excludeUserIds: [IDs from segment 1] → remaining users
Segment 3: all users, excludeUserIds: [IDs from segments 1+2] → remaining users
```

### Step 3: Auto-exclude isInternal users

**DOUBLE EXCLUSION (belt and suspenders):**

1. **MCP level:** The `send_targeted_email` tool should filter isInternal users. But dont rely on this alone.
2. **Skill level:** Before building the recipient list, explicitly filter out any users where `isInternal: true`. These are persona/staff accounts and must NEVER receive campaign emails.

Call `get_self_personas` MCP to get the list of internal persona IDs. Add all of them to the excludeUserIds list.

### Step 4: Build the email

**Subject line rules:**
- Social proof subjects win: "3 posts your fellow developers are discussing" > "New posts on Webmatrices"
- Personalize with segment context: "Your AdSense RPM might be affected" for AdSense segment
- No clickbait. No ALL CAPS. No excessive punctuation.
- Under 60 characters

**Body rules:**
- Max 2-3 posts per email. More than that = nobody clicks any.
- Max 1 link per paragraph.
- Inline links with arrows: `<a href="url">Post title →</a>`
- Brief 1-2 sentence teaser per post, NOT the full excerpt
- Personal tone from the site owner (first person, lowercase)

**CTA button with Gmail white text fix:**

Gmail sometimes renders white text on white buttons. Always wrap CTA button text in a `<span>` with explicit color:

```html
<a href="https://webmatrices.com/post/slug-here"
   style="display: inline-block; padding: 12px 24px; background: #9333ea;
          border-radius: 6px; text-decoration: none; font-weight: 600;">
  <span style="color: #ffffff;">Read the discussion →</span>
</a>
```

**Email structure:**

```html
<p>hey [segment-relevant greeting],</p>

<p>[1-2 sentence context about why this email matters to them]</p>

<p><strong>[Post 1 title]</strong></p>
<p>[1-2 sentence teaser that creates curiosity, not summary]</p>
<p><a href="https://webmatrices.com/post/[slug]">[title] →</a></p>

<p><strong>[Post 2 title]</strong></p>
<p>[1-2 sentence teaser]</p>
<p><a href="https://webmatrices.com/post/[slug]">[title] →</a></p>

[CTA button]

<p>- [site owner's first name, fetch from MCP]</p>
```

### Step 5: Present the DRAFT

Show the complete email:
- Subject line
- Target segment(s) with estimated user counts
- Excluded segments/users
- Full HTML body
- STATE: DRAFT

Ask: "Ready to preview? Run `/campaign preview` to send a test to the site owner."

**NEVER auto-send. NEVER proceed past DRAFT without user action.**

---

## SUBCOMMAND: /campaign preview

1. Send the email to the site owner ONLY using `send_email` MCP with:
   - `userId: 1` (the site owner's user ID)
   - Full subject and body as built in the draft
   - Do NOT use `send_targeted_email` for preview — it picks from the segment, not the site owner
3. Move state to PREVIEWED
4. Tell the user: "Test email sent to the site owner. Check your inbox. When ready: `/campaign send`"

---

## SUBCOMMAND: /campaign send

1. Check if state is PREVIEWED
   - If YES: proceed to send
   - If NO: warn "This campaign hasn't been previewed. Are you sure you want to send to [N] users without testing? Say 'yes send it' to confirm."
2. Send via `send_targeted_email` MCP:
   - Use segment targeting with excludeUserIds chaining
   - Ensure isInternal users are excluded
3. Move state to SENT
4. Report: "Campaign sent to [N] users across [segments]. Subject: [subject]"

---

## SAFETY CHECKLIST

Before ANY send (preview or full):

- [ ] All post slugs verified to exist (no broken links)
- [ ] isInternal users excluded at BOTH MCP and skill level
- [ ] excludeUserIds chaining applied for multi-segment sends
- [ ] Subject line under 60 characters
- [ ] Max 2-3 posts in the email
- [ ] Max 1 link per paragraph
- [ ] CTA button has span wrapper for Gmail white text fix
- [ ] No competitor links in the email
- [ ] Personal tone (from the site owner, not corporate)
- [ ] Preview sent to the site owner before full send (or explicit override)

---

## COMMON CAMPAIGN TYPES

### Post promotion
Promote 2-3 recent posts to relevant segments. Most common campaign.

### Welcome sequence
New user just signed up. Send a welcome email with top 3 posts to get them engaged.

### Re-engagement
Users who haven't visited in 30+ days. "Here's what you missed" with top posts.

### Announcement
New feature, new tool, platform update. Goes to all users (except isInternal).

---

## IMPORTANT

- State machine is non-negotiable. DRAFT -> PREVIEWED -> SENT.
- NEVER send to full audience without at least offering preview
- Always verify post slugs exist before building email
- Always exclude isInternal users (persona accounts)
- Always chain excludeUserIds when targeting multiple segments
- CTA buttons MUST use span wrapper for Gmail compatibility
- Max 2-3 posts per email. Information overload = zero clicks.
