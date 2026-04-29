# webmatrices-skills

Claude Code plugin for managing the [Webmatrices](https://webmatrices.com) community platform.

## Skills

### Content creation

| Skill | Command | Description |
|-------|---------|-------------|
| trending | `/webmatrices:trending [topic]` | Multi-platform content discovery across Reddit, Twitter, Dev.to, Medium, and Google News with viral scoring |
| write | `/webmatrices:write [persona] [topic]` | Write and publish posts, replies, comments, and HN replies as a persona via MCP |
| fact-check | `/webmatrices:fact-check [postId/slug/text]` | Verify factual claims against primary sources before publishing |
| campaign | `/webmatrices:campaign [recent/postId/description]` | Email campaign workflow with DRAFT/PREVIEWED/SENT safety states, backed by Brevo performance data |

### Engagement

| Skill | Command | Description |
|-------|---------|-------------|
| simulate-engagement | `/webmatrices:simulate-engagement [post-slug]` | Orchestrate organic-looking engagement with timing, persona rotation, and comment diversity |
| smell | `/webmatrices:smell [postId/slug/commentId/username/text]` | Universal authenticity scanner for AI tells, factual issues, and cross-persona bleeding |
| reduce-smell | `/webmatrices:reduce-smell [postId/slug/commentId/text]` | Generate diff-style fixes for problems found by `/smell` (preview only, never publishes) |

### Moderation & admin

| Skill | Command | Description |
|-------|---------|-------------|
| spam-scanner | `/webmatrices:spam-scanner` | Scan for spam posts, soft-delete them, ban spammers |
| ban-user | `/webmatrices:ban-user <id>` | Ban a user by ID, username, or email |
| create-persona | `/webmatrices:create-persona <username>` | Create a persona account for content seeding |
| find | `/webmatrices:find <query>` | Search posts, users, tags, and analytics with natural language |
| audience-matcher | *(auto-invoked)* | Match a topic to the best persona by voice, topic, backstory, and fatigue |

## Install

Add the marketplace, then install:

```bash
/plugin marketplace add Bishwas-py/webmatrices-skills
/plugin install webmatrices-skills@webmatrices
```

Or load locally without installing:

```bash
claude --plugin-dir /path/to/webmatrices-skills
```

### Team setup

Add to your project's `.claude/settings.json` so team members get prompted automatically:

```json
{
  "extraKnownMarketplaces": {
    "webmatrices-skills": {
      "source": {
        "source": "github",
        "repo": "Bishwas-py/webmatrices-skills"
      }
    }
  },
  "enabledPlugins": {
    "webmatrices-skills@webmatrices": true
  }
}
```

## Requirements

- Webmatrices project with Prisma client generated
- Database connection string (via `.env` or provided directly)
