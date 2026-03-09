---
name: trending-topics
description: Fetch trending topics from the internet relevant to Webmatrices audience. Use when asked about trending topics, what to write about, content ideas, or what's hot.
disable-model-invocation: true
---

# Trending Topics

Fetch and curate trending topics from the internet that are relevant to the Webmatrices audience.

## Webmatrices Audience Segments

The platform serves these key audiences (ordered by size):

1. **AdSense seekers** — people trying to get approved or improve ad revenue
2. **Fiverr freelancers** — sellers looking to improve gig performance
3. **SEO practitioners** — from beginners to agency owners
4. **Web developers** — SvelteKit, AI tools, vibe coding, SaaS builders
5. **Startup founders** — indie hackers, micro-SaaS, bootstrappers

## Search Queries to Run

Use WebSearch with queries like:
- `Google AdSense changes [current month] [current year]`
- `Google core update [current month] [current year]`
- `Fiverr trends freelancing [current year]`
- `AI coding tools vibe coding news [current month] [current year]`
- `SEO news changes [current month] [current year]`
- `trending tech discussions Reddit Hacker News [current month] [current year]`
- `SaaS startup news [current month] [current year]`

## Output Format

For each relevant topic, provide:
- **Topic title** — what happened
- **Why it matters for Webmatrices** — which audience segment cares
- **Post angle** — how to write about it (what persona, what hook)
- **Urgency** — is it time-sensitive (deadline, rollout date)?

## Workflow

1. Run 3-5 WebSearch queries covering the audience segments
2. Filter results for relevance to Webmatrices audience
3. Rank by: time-sensitivity > audience size > engagement potential
4. Present 5-10 curated topics with actionable post angles
5. Recommend the top 2-3 to write about first
