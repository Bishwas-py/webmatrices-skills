---
name: find
description: Search posts, users, tags, and analytics on the Webmatrices database. Use when asked to find posts, search users, check engagement, look up tags, or query platform data.
disable-model-invocation: true
argument-hint: [natural-language-query]
---

# Find

Search and query the Webmatrices production database using MCP tools.

## Available MCP Tools

| Tool | Use For |
|------|---------|
| `list_posts` | Browse posts with search, filters, pagination |
| `search_posts_by_content` | Find posts containing specific text in body |
| `list_users` | Search users by username/email, filter by status |
| `get_user_details` | Full profile for a specific user |
| `list_tags` | Browse tags with post counts |
| `get_stats` | Platform-wide statistics |

## Query Detection

Parse the natural language input and map to the right MCP tool(s):

### Post queries

| Input pattern | MCP tool | Parameters |
|---------------|----------|------------|
| `"fiverr posts"`, `"posts about adsense"` | `list_posts` | `search: "fiverr"` |
| `"posts containing whatsapp"` | `search_posts_by_content` | `pattern: "whatsapp"` |
| `"recent posts"`, `"latest posts"` | `list_posts` | `sortBy: "createdAt"` |
| `"deleted posts"` | `list_posts` | `filter: "deleted"` |
| `"posts by user [userId]"` | `list_posts` | `authorId: [userId]` |

### User queries

| Input pattern | MCP tool | Parameters |
|---------------|----------|------------|
| `"find user john"` | `list_users` | `search: "john"` |
| `"banned users"` | `list_users` | `filter: "banned"` |
| `"staff users"` | `list_users` | `filter: "staff"` |
| `"user #12345"` | `get_user_details` | `userId: 12345` |
| `"user @username"` | `get_user_details` | `username: "username"` |

### Tag queries

| Input pattern | MCP tool | Parameters |
|---------------|----------|------------|
| `"all tags"`, `"top tags"` | `list_tags` | |
| `"tag adsense"` | `list_tags` | `search: "adsense"` |

### Analytics

| Input pattern | MCP tool |
|---------------|----------|
| `"how many users"`, `"platform stats"` | `get_stats` |

### Combining filters

Queries can combine multiple MCP calls:
- `"banned users with posts"` → `list_users` (filter: banned) + `list_posts` (per user)
- `"spam posts containing brimo"` → `search_posts_by_content` (pattern: "brimo")

## Output Format

Present results as clean markdown tables:

### Posts
```
| # | Title | Author | Tags | Views | Comments | Age |
```

### Users
```
| # | ID | Username | Email | Active | Posts | Comments | Joined |
```

### Always include
- Total count at the bottom: `**Found 23 posts matching "fiverr"**`
- Default limit: 20 results. Show more if user asks.

## Example Queries

```
/webmatrices:find "fiverr posts"
/webmatrices:find "banned users"
/webmatrices:find "posts containing whatsapp"
/webmatrices:find "user @[username]"
/webmatrices:find "platform stats"
/webmatrices:find "top tags"
```
