---
name: create-persona
description: Create a new user persona account for content seeding on Webmatrices. Use when asked to create a new user, persona, or fake account for posting.
disable-model-invocation: true
argument-hint: [username]
---

# Create Persona

Create a new user account on Webmatrices for content seeding purposes.

> **Note:** There is no MCP tool for user creation yet. This skill uses a temporary database script. All other operations (posting, replying, banning) should use MCP tools.

## Database Connection

All scripts MUST run from `~/Projects/webm-frontend` for Prisma access. Read production credentials from `.env`:

```bash
cd ~/Projects/webm-frontend
DATABASE_URL=$(grep '^PROD_DATABASE_URL=' .env | cut -d'"' -f2) node temp-script.mjs
```

## Password Hashing

Use the Django PBKDF2 format used by the codebase:

```typescript
import crypto from 'crypto';

function hashDjangoPassword(password: string): string {
  const iterations = 390000;
  const salt = crypto.randomBytes(12).toString('base64');
  const hash = crypto.pbkdf2Sync(password, salt, iterations, 32, 'sha256');
  return `pbkdf2_sha256$${iterations}$${salt}$${hash.toString('base64')}`;
}
```

## Persona Guidelines

When creating a persona, match these conventions:
- **Email**: Use `bishwasbh+<username>@gmail.com` format
- **Password**: Use a strong default like `Webm@tr1ces2026!` (tell the user)
- **Bio**: Short, lowercase, casual — matches the community tone
- **Auth status**: Set `isConfirmed: true` so they can post immediately

## Existing Persona Styles (reference)

Fetch existing personas from `get_self_personas` MCP to check for voice/topic overlap before creating a new one. Each persona includes `metadata.personaTraits` with voice, topics, opinion strength, and backstory. Use this data to ensure the new persona fills a gap rather than duplicating an existing voice or topic area.

## Workflow

1. If username provided as argument, use it. Otherwise, suggest 3-4 username options
2. Ask the user to pick or provide a username
3. Create the user with a temporary script:
   - `isActive: true`, `isStaff: false`, `isSuperuser: false`
   - A Profile with a short bio
   - An AuthStatus with `isConfirmed: true`
4. Display the created user details (ID, username, email, password)
5. Clean up temporary scripts
6. After creation, use MCP tools for all further operations (posting, etc.)
