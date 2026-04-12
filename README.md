# webmatrices-skills

Claude Code plugin for managing the [Webmatrices](https://webmatrices.com) community platform.

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| viral-writer | `/webmatrices:viral-writer [persona] [topic]` | Write viral content and publish it as a persona via MCP |
| simulate-engagement | `/webmatrices:simulate-engagement [post-slug]` | Orchestrate organic-looking engagement with timing, persona rotation, and comment diversity |
| detect-fake-engagement | `/webmatrices:detect-fake-engagement [scope]` | Audit existing engagement for AI tells, timing anomalies, and suspicious patterns |
| trending-topics | `/webmatrices:trending-topics` | Fetch trending topics for the audience |
| spam-scanner | `/webmatrices:spam-scanner` | Scan for spam posts, soft-delete them, ban spammers |
| ban-user | `/webmatrices:ban-user <id>` | Ban a user by ID, username, or email |
| create-persona | `/webmatrices:create-persona <username>` | Create a persona account for content seeding |
| audience-matcher | *(auto-invoked)* | Match topics to the best persona |
| find | `/webmatrices:find <query>` | Search posts, users, tags, and analytics with natural language |

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
