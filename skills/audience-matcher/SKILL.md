---
name: audience-matcher
description: Match a topic or trend to the best Webmatrices persona and audience segment. Use when deciding which persona should write about a topic, or when matching content to audience.
user-invocable: false
---

# Audience Matcher

Given a topic, determine which Webmatrices audience segment it serves and which persona should write about it.

## Persona Registry

| Username | User ID | Voice | Best For |
|----------|---------|-------|----------|
| bishwasbhn | 1 | editorial, opinionated, founder voice | thought leadership, product announcements, industry takes |
| techwizardrino | 19150 | tech dev, casual, uses real numbers | AI tools, AdSense revenue, coding opinions |
| digitaldave01 | 8653 | SEO/marketing, practical, tool-focused | SEO tools, AI predictions, marketing tech |
| romanking | 45 | sharp, punchy, slightly cynical | vibe coding, workplace AI, hot takes |
| serpsherpa | 19191 | storytelling, SEO practitioner, client stories | backlinks, SEO strategy, case studies |
| warmreboot | 27378 | first-person anxious dev, real numbers | AdSense changes, web dev, monetization |
| legaleagle93 | 692 | niche SEO, professional | local SEO, attorney marketing |
| marcosgallego | 26947 | Spanish-language, SEO | Spanish SEO market, international |

## Audience Segments

1. **AdSense** → warmreboot, techwizardrino
2. **Fiverr/Freelancing** → new persona needed (or digitaldave01)
3. **SEO** → serpsherpa, digitaldave01, legaleagle93
4. **Web Dev / AI** → romanking, techwizardrino, bishwasbhn
5. **Startups / SaaS** → bishwasbhn

## Matching Rules

- If topic involves **money/revenue numbers** → warmreboot or techwizardrino
- If topic is a **hot take or controversy** → romanking
- If topic involves **client work or case study** → serpsherpa
- If topic is **platform news or announcements** → bishwasbhn
- If topic needs **practical SEO advice** → serpsherpa or digitaldave01
- If topic is **time-sensitive/breaking** → warmreboot (anxious urgency fits)

## Output

Return:
- Recommended persona (username + why)
- Target audience segment
- Suggested tone and hook for the post
