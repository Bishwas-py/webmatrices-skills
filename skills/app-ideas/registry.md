# App Ideas Registry

Canonical state of researched, published, killed, and queued app ideas. Lives in the skill so it survives memory wipes. Refresh from `mcp__webmatrices__list_app_ideas` periodically and after any `/app-ideas publish` run.

**Last verified:** 2026-04-29 (28 published)

---

## Published — credible (12+ pain points each)

Use `update_app_idea` + `add_*` / `update_*` granular tools to modify these. Never re-call `create_app_idea` (destructive — wipes all relations).

| Slug | Category | Pain | Build | Comp | PP | Sol | Upvotes |
|---|---|---|---|---|---|---|---|
| `ada-litigation-shield` | legal-compliance | 9 | 6 | 3 | 15 | 6 | 2,858 |
| `ecom-profit-tracker` | ecommerce | 9 | 8 | 5 | 16 | 5 | 1,357 |
| `simple-expense-tracker` | finance-ops | 8 | 5 | 7 | 14 | 4 | 3,384 |
| `appointment-text-reminder` | healthcare-ops | 8 | 5 | 7 | 14 | 4 | 144,527 |
| `cottage-food-legality-checker` | small-business-compliance | 8 | 5 | 3 | 16 | 6 | 149 |
| `grant-clock` | nonprofit-tools | 8 | 4 | 3 | 15 | 8 | 144 |
| `freelancer-client-shield` | freelance-tools | 8 | 5 | 5 | 12 | 5 | 68,246 |
| `layoff-early-warning-dashboard` | career-tools | 8 | 6 | 2 | 14 | 3 | 84,245 |
| `freelancer-client-pipeline` | freelance-tools | 8 | 6 | 5 | 14 | 4 | 2,994 |
| `residential-electrician-estimator` | trades-tools | 8 | 6 | 3 | 15 | 7 | 48 |
| `client-content-collector` | freelance-tools | 7 | 3 | 3 | 13 | 4 | 54,938 |
| `restaurant-menu-sync` | restaurant-tech | 7 | 8 | 5 | 13 | 3 | 31,147 |
| `bookkeeper-doc-chaser` | accounting-tools | 7 | 5 | 5 | 15 | 4 | 1,222 |
| `invoice-escalator` | freelance-tools | 7 | 3 | 3 | 14 | 4 | 2,270 |
| `portable-freelancer-reputation` | freelance-tools | 7 | 5 | 3 | 14 | 3 | 6,457 |
| `accessible-prenup-builder` | legal-tech | 7 | 6 | 4 | 13 | 3 | 23,327 |
| `streaming-royalty-tracker` | fintech | 7 | 6 | 3 | 13 | 3 | 6,751 |
| `privacy-local-document-toolkit` | privacy-tools | 7 | 4 | 6 | 17 | 5 | 20,913 |
| `order-tracking-responder` | ecommerce | 7 | 8 | 5 | 12 | 4 | 5,396 |
| `changelog-release-notes-automator` | dev-tools | 6 | 5 | 5 | 16 | 4 | 84 |
| `service-pricing-calc` | service-business | 6 | 3 | 3 | 13 | 3 | 1,540 |
| `inspection-translator` | real-estate | 6 | 5 | 1 | 13 | 3 | 3,355 |
| `contract-tracker` | business-ops | 6 | 5 | 3 | 13 | 4 | 2,369 |
| `review-aggregator` | ecommerce | 5 | 5 | 5 | 13 | 4 | 14,848 |
| `seo-agency-finder` | seo-tools | 5 | 5 | 7 | 14 | 3 | 2,466 |
| `ai-file-renamer` | seo-tools | 5 | 3 | 3 | 13 | 3 | 78,444 |

## Published — thin (consider deleting via `delete_app_idea`)

| Slug | Category | Pain | Comp | PP | Reason |
|---|---|---|---|---|---|
| `keyword-rank-checker` | seo-tools | 4 | 9 | 1 | Saturated SEO category, low pain. Not worth enriching. |
| `backlink-checker` | seo-tools | 3 | 9 | 1 | Same — saturated, low pain. |

---

## KILLED hypotheses (DO NOT re-research)

When `track` surfaces these names again, skip Round 1 entirely.

| Hypothesis | Killed | Why |
|---|---|---|
| Employer Brand Monitor for SMBs | 2026-04-29 | SMB pain is Google reviews not Glassdoor; Birdeye/LocalClarity own from $8/mo. |
| Meeting-to-Action-Item Bridge | 2026-04-29 | Fellow.ai ships exact wedge at $7/user; Otter has Jira push native. Brutally crowded. |
| Impulse Buy Preventer (Chrome Extension) | 2026-04-29 | 12+ free extensions exist (Checkout Chill is literal MVP). Target users in debt won't pay. |
| Public Records & Lobbying Investigator | 2026-04-29 | Paying audience owned by Quorum/FiscalNote/BGov; journalists won't pay sustained subs. TAM too small for B2C. |
| B2B Lead Stack Unifier | 2026-04-29 | Hypothesis was wrong — Clay IS the unifier. RingLead/Insycle/Cloudingo cover all price points. |

---

## Adjacent wedges surfaced (worth Round 1 if revisited)

These came up during scouting for KILLED ideas. Real signal, not yet validated.

| Wedge | Source | Notes |
|---|---|---|
| Local journalism PDF watcher | r/Journalism scout 2026-04-29 | AI agent that watches city council / school board PDF dumps and pings local reporters with story leads. Substack stringers may pay $20-50/mo. |
| No-bot local meeting recorder + on-device action items | r/ProductivityApps scout 2026-04-29 | Bluedot's gap. Unmet ask was "no bot in the call" + "built-in task manager I don't have to leave." |
| Slack-native "what did we decide about X" retrieval | r/IMadeThis scout 2026-04-29 | Pull-side of meeting notes — the unsolved half. |

---

## Categories already well-covered (be skeptical of new ideas here)

- `freelance-tools` (5 ideas)
- `ecommerce` (3 ideas)
- `seo-tools` (3 ideas — and 2 are thin)
- `legal-compliance` + `legal-tech` (2 ideas)

Round 1 should weight competition higher in these categories.

---

## How to keep this file accurate

After any `/app-ideas publish` or `/app-ideas track` session:

1. Add new published slugs to the credible table with their scores
2. Add KILLED hypotheses with one-line reasons (saves future Round 1)
3. Add adjacent wedges surfaced during scouting
4. Bump the "Last verified" date
5. Mirror the changes into user memory `project_app_idea_enrichment.md` so they're available cross-session

Refresh source-of-truth via:

```
mcp__webmatrices__list_app_ideas(limit=50, sortBy='painIntensity', sortOrder='desc')
```

The MCP returns counts and scores per idea — paste into the credible table.
