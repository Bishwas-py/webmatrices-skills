---
name: fact-check
description: Verify factual claims in Webmatrices content against primary sources. Checks study references, product pricing, company actions, statistics, and platform behavior claims. Use when asked to fact-check, verify claims, check accuracy, or validate content.
disable-model-invocation: true
argument-hint: [postId/slug/"raw text"]
---

# Fact-Check

Verify factual claims in Webmatrices content against primary sources. This skill exists because HN tore apart a post (2026-04-21) for getting Vercel's encryption model wrong. Wrong facts destroy credibility. Spicy opinions are fine. Wrong facts are not.

Called automatically by `/write` before presenting drafts. Can also be run standalone.

---

## INPUT

| Input | What it does |
|-------|-------------|
| Post ID | Fetches post via `get_post` MCP, extracts and checks claims |
| Post slug | Fetches via `get_post_by_slug` MCP |
| Quoted text ("...") | Scans raw text for claims to verify |
| No args | Error -- requires input |

---

## CLAIM TYPES TO EXTRACT

Parse the content and extract every verifiable claim. Categorize each one:

| Claim type | Example | What to verify against |
|-----------|---------|----------------------|
| Study reference | "a Stanford study found that 75%..." | Find the actual paper. Link to it. |
| Product pricing | "costs $20/month" | Check the product's current pricing page |
| Company action | "Google announced deprecation of..." | Find the official announcement |
| Statistic | "75% of developers use AI tools" | Find the original survey/report |
| Platform behavior | "Vercel encrypts env vars at rest" | Check official docs at vercel.com/docs |
| Version/release date | "released in March 2026" | Check official changelog/blog |
| API behavior | "the API returns a 429 after 100 requests" | Check official API docs and rate limit pages |
| Policy claim | "Google requires 30 articles for AdSense" | Check official AdSense policies |
| Legal/regulatory | "GDPR requires consent for..." | Check official regulation text |

### What to SKIP

- **Hypothetical persona numbers.** "I made $47.23 last month" is the persona's own experience claim -- dont fact-check it.
- **Opinions.** "I think backlinks are dead" is a take, not a fact.
- **Predictions.** "AI will replace 50% of content writers by 2028" is speculation.
- **Named patterns.** "The Control Tax" is a framework the persona created -- its definitionally correct.
- **Subjective observations.** "My RPM tanked" is personal experience.

---

## VERIFICATION WORKFLOW

For each extracted claim:

### Step 1: Search primary sources

Use `WebSearch` to find the **primary source**, not secondary coverage.

**Primary source hierarchy (in order of preference):**
1. Official documentation (product docs, API docs)
2. Official announcements (company blogs, press releases)
3. Published papers (arXiv, journal sites, conference proceedings)
4. Official data sources (FRED, BLS, Census, Google Transparency Report)
5. Regulatory text (legislation.gov, eur-lex, FTC)

**NOT primary sources:**
- Blog posts about the product (someone's interpretation)
- Reddit comments
- Twitter threads
- News articles quoting unnamed sources
- "According to reports"

### Step 2: Compare claim against source

For each claim, determine the verdict:

| Verdict | Meaning | Action |
|---------|---------|--------|
| VERIFIED | Claim matches primary source exactly | Keep as-is. Add source link. |
| STALE | Claim was true but is outdated | Update with current information. Note what changed. |
| UNVERIFIED | Cannot find primary source to confirm or deny | Reframe as persona observation or remove. |
| WRONG | Primary source contradicts the claim | Must fix. Either correct the fact or reframe as persona's misunderstanding + correction. |

### Step 3: Special rules by claim type

**Pricing claims: ALWAYS check CURRENT pricing.**
- Products change pricing frequently
- Go to the actual pricing page, not a review site
- Note the plan tier if relevant (free vs. pro vs. enterprise)
- If pricing recently changed, that itself is interesting content

**Study references: LINK to the actual paper.**
- Search for the paper title on Google Scholar, arXiv, or the publisher
- Verify the claim matches what the paper actually says (not what someone said the paper says)
- Check the sample size and methodology if the claim is about a percentage
- "A study found 75%..." is meaningless without knowing N=50 vs N=50,000

**Company actions: FIND the official announcement.**
- Search for "[company] blog [action]" or "[company] announcement [topic]"
- Press releases and official blog posts are primary sources
- News articles citing "sources familiar" are NOT primary sources
- If the announcement cant be found, the claim might be rumor

**Platform behavior: CHECK official docs, not community lore.**
- "Vercel stores env vars in plaintext" -- check vercel.com/docs, not a dev blog
- "AWS S3 buckets are public by default" -- check AWS docs (this one changed over the years)
- Platform behavior changes. What was true in 2024 may not be true in 2026.

---

## OUTPUT FORMAT

```
FACT-CHECK REPORT
═════════════════

Source: [post title / raw text preview]
Claims found: 7
Verified: 4 | Stale: 1 | Unverified: 1 | Wrong: 1

───────────────

CLAIM 1: "Vercel encrypts environment variables at rest"
Type: platform-behavior
Verdict: VERIFIED
Source: https://vercel.com/docs/environment-variables
Note: Vercel encrypts env vars at rest using AES-256. They are decrypted at build time and runtime.

CLAIM 2: "costs $20/month for the pro plan"
Type: pricing
Verdict: STALE
Source: https://vercel.com/pricing
Current: Pro plan is now $20/user/month (changed from flat $20/month in Jan 2026)
Fix: Update to "$20/user/month" and note the per-user pricing model

CLAIM 3: "a Stanford study found that 75% of developers use AI tools"
Type: study-reference
Verdict: UNVERIFIED
Note: Cannot find this specific Stanford study. There IS a Stack Overflow 2025 survey showing 76% of developers use AI tools. Consider citing that instead with a link.
Fix: Replace with "the 2025 Stack Overflow survey found 76% of developers use AI coding tools" + link

CLAIM 4: "Google requires sites to have 30 articles for AdSense approval"
Type: policy
Verdict: WRONG
Source: https://support.google.com/adsense/answer/9724
Note: Google has NO official minimum article count. The 30-article rule is community lore. Official requirements focus on sufficient content, original content, and policy compliance.
Fix: Remove the specific number. Say "enough original content to demonstrate value" or reframe as the persona's own experience ("in my experience sites with fewer than 30 posts tend to get rejected but google doesnt officially state a minimum")

───────────────

SUMMARY:
- 1 claim needs correction (Claim 4: wrong)
- 1 claim needs updating (Claim 2: stale pricing)
- 1 claim needs better sourcing (Claim 3: unverified study)
- Add source links for verified claims where missing
```

---

## WHEN CALLED BY /write

When `/write` calls this skill internally during Step 3.5:

1. Extract all verifiable claims from the draft
2. Run verification on each
3. Return results to `/write` so it can fix the draft before presenting to the user
4. If any claims are WRONG, the draft MUST be corrected before presenting
5. If claims are STALE, update the numbers
6. If claims are UNVERIFIED, either find a better source or reframe as persona observation

The user should see a clean draft with all facts already verified. The fact-check step is invisible to the user unless there are unresolvable issues.

---

## WHEN CALLED STANDALONE

When the user runs `/fact-check` directly:

1. Show the full report with all claims and verdicts
2. For WRONG and STALE claims, suggest specific rewrites
3. For UNVERIFIED claims, suggest alternative sources or reframings
4. Include links to all primary sources found

---

## IMPORTANT

- NEVER trust secondary sources over primary sources
- NEVER accept a blog post's interpretation of official docs without checking the docs
- NEVER accept Reddit/HN corrections without verifying them too
- Pricing = ALWAYS check current. Products change pricing constantly.
- Studies = LINK to the actual paper. "A study found" without a link is meaningless.
- If a claim cannot be verified, the safe move is to reframe it as the persona's own observation, not state it as fact
