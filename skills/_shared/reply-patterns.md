# Reply Patterns — What Gets Engagement on Webmatrices

Data from analysis of 359 comments across 175 active posts.

## Key Stats

- Admin/persona accounts produce 56% of all comments (202/359)
- Top-liked replies average 2-5 likes
- Zero-engagement replies: generic advice, "thanks for sharing", link-only

## Patterns That Work

### 1. "I actually looked at YOUR thing"
Reference the user's specific site, gig, numbers, or situation — not generic advice.

**Example (3 likes):**
> i looked at your fiverr gig — your title says "i will design a logo" which is literally what 47,000 other sellers say. your 4.8 rating is solid but your thumbnail looks like a canva template...

### 2. Lead with what's working
Acknowledge something positive before any criticism. People engage more when they don't feel attacked.

**Example (4 likes):**
> level 2 seller, 24 reviews, 4.9 rating — your fundamentals are solid. the problem isn't your skills, it's your positioning...

### 3. One contrarian insight
Include something unexpected that challenges conventional wisdom.

**Example (3 likes):**
> everyone says "post more content" but honestly your site has enough articles. the real problem is your internal linking — google can't find half your pages...

### 4. Specific data and numbers
Exact numbers, percentages, and comparisons feel more credible than vague statements.

**Example (5 likes):**
> your site loads in 6.2 seconds on mobile. google's threshold is 2.5s. that alone is probably why you got rejected — adsense bots crawl on simulated 3G...

### 5. End with a direct question
Questions drive follow-up replies and keep the conversation going.

**Example (3 likes):**
> ...what's your current click-through rate on search console? that'll tell us if it's a ranking problem or a snippet problem.

### 6. Conversational, lowercase tone
Never sound like ChatGPT. Use first person, contractions, slight frustration/urgency.

**Good:** "nobody tells you this but adsense hates single-page sites"
**Bad:** "I'd be happy to help! Here's a comprehensive guide to AdSense approval..."

## Patterns That Fail (0 engagement)

| Pattern | Example | Why It Fails |
|---------|---------|--------------|
| Generic advice list | "1. Create quality content 2. Improve SEO 3. Be patient" | Could apply to anyone |
| Empty acknowledgment | "thanks for sharing this" | Adds nothing |
| ChatGPT voice | "Great question! Here's a comprehensive overview..." | Feels robotic |
| Link-only | "check this: [link]" | No personal insight |
| Placeholder | "looking into it, will update" | Never follows up |
| Over-enthusiasm | "This is AMAZING! You're doing GREAT!" | Feels fake |

## Persona Voice Characteristics

Fetch persona voice data from `get_self_personas` MCP tool. Each persona's `metadata.personaTraits` contains their voice style, opinion strength, topics, and writing samples. Match the reply voice to the persona's traits from MCP, not from a static table.

**General voice categories for replies:**
- Direct admin authority voices work best for site reviews and technical fixes
- Anxious dev energy voices fit revenue issues and relatable problems
- Deep audit / methodical voices suit detailed analysis and gig reviews
- Buyer perspective voices work for practical tips and advice
- SEO storyteller voices suit strategy discussions and case studies
- Sharp contrarian voices fit hot takes and opinion discussions

## Reply Length Guidelines

| Context | Target Length |
|---------|-------------|
| Site/gig review (URL provided) | 300-500 words |
| General advice reply | 150-300 words |
| Discussion/opinion reply | 100-200 words |
| Quick follow-up | 50-100 words |

## HTML Format

All replies use HTML tags: `<p>`, `<strong>`, `<ol>`, `<ul>`, `<code>`, `<a>`.
No markdown — the platform renders HTML via TipTap editor output.
