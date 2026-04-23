---
name: audience-matcher
description: Match a topic or trend to the best Webmatrices persona and audience segment. Fetches personas from MCP, considers voice fit, topic relevance, backstory, recency, and fatigue. Use when deciding which persona should write about a topic, or when matching content to audience.
user-invocable: false
---

# Audience Matcher

Given a topic, determine which Webmatrices audience segment it serves and which persona should write about it. All persona data is fetched live from MCP, never hardcoded.

## Persona Data -- ALWAYS FROM MCP

**NEVER hardcode persona usernames, IDs, voice descriptions, or topic mappings.**

At the start of every audience-matcher invocation:

1. Call `get_self_personas` MCP tool to fetch all personas
2. Each persona includes `metadata.personaTraits` with: voice, topics, opinion strength, backstory
3. Use these traits for matching, not a static table

## Matching Algorithm

For a given topic, score each persona on five factors:

### 1. Topic Relevance (weight: 35%)

Compare the topic against each persona's `topics` field from their traits:
- Direct match (topic is listed in persona's topics) = 1.0
- Adjacent match (topic is related to persona's topics) = 0.5
- No match = 0.0

### 2. Voice Fit (weight: 25%)

Does the topic suit the persona's voice style?
- Hot takes and controversies fit sharp/punchy voices
- Data-heavy topics fit methodical/data-first voices
- Confessional topics fit vulnerable/anxious voices
- Strategic topics fit calm/veteran voices

### 3. Backstory Relevance (weight: 15%)

Does the persona have a plausible backstory connection to this topic?
- A persona who's established as a web dev cant credibly write about advanced SEO case studies
- A persona who focuses on monetization cant credibly write about cutting-edge ML research
- Check `backstory` field in persona traits for relevant experience

### 4. Recency Check (weight: 15%)

How recently has this persona posted?
- Fetch last 3 posts via `list_posts` with `authorId`
- If persona posted in the last 24 hours = deprioritize (fatigue risk)
- If persona posted about a SIMILAR topic in the last 7 days = deprioritize (repetition)
- If persona hasnt posted in 5+ days = boost (they're "due")

### 5. Persona Fatigue (weight: 10%)

Track how often a persona has been used recently:
- Same persona 3 days in a row = deprioritize heavily
- Same persona every other day for a week = moderate deprioritization
- Persona used once this week = no penalty

## Audience Segments

Match the topic to audience segments based on content:

| Segment | Topic Indicators |
|---------|-----------------|
| AdSense | AdSense, RPM, CPC, ad revenue, ad approval, Google ads |
| SEO | Backlinks, rankings, search console, SERP, keywords, indexing |
| Web Dev / AI | Coding, AI tools, vibe coding, LLMs, deployment, frameworks |
| Freelancing | Fiverr, Upwork, freelance, gig economy, client work |
| Startups / SaaS | Product, startup, pricing, MRR, churn, launch |
| Content / Blogging | Content strategy, blogging, writing, publishing |
| GEO / AEO | AI search, AI citations, AI overviews, GEO optimization |

## Output

Return:
- **Recommended persona:** username + ID (from MCP data) + why they're the best fit
- **Runner-up persona:** in case the primary is fatigued or recently used
- **Target audience segment**
- **Suggested tone and hook** for the post (based on persona voice + topic)
- **Recency note:** when the recommended persona last posted and about what

## Example

```
MATCH RESULT:
Topic: "AdSense RPM tanking after March 2026 core update"

Primary: [matched persona from MCP] (ID from MCP)
  Score: 0.91
  Why: [topic] is in their topics array, [voice description] fits the topic's tone,
       hasn't posted in 3 days, backstory includes relevant experience

Runner-up: [second-best match from MCP] (ID from MCP)
  Score: 0.72
  Why: [adjacent topic] is in their topics array, but [voice description]
       is less urgent than the topic demands

Segment: AdSense
Tone: anxious urgency, personal numbers, "is anyone else seeing this?"
Hook: specific RPM drop with exact numbers from "his own dashboard"
```

## IMPORTANT

- ALL persona data comes from `get_self_personas` MCP. Zero hardcoded personas.
- Always check recency before recommending. A persona who just posted yesterday is a weaker choice.
- Consider persona fatigue. Rotating personas keeps the platform looking organic.
- The runner-up recommendation exists because the primary might have just posted. Give the caller a backup option.
