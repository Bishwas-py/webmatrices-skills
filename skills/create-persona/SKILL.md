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

## Persona Traits Schema (v2.1)

The `metadata.personaTraits` JSON blob on the User table holds the persona's voice, perspective, and writing patterns. There is no strict schema (JSON column), but the following keys are conventional and consumed by `/write-post`, `/write-reply`, `/smell`, and `/reduce-smell`.

### Voice layer (drives sentence shape)

- `writingSamples` — the load-bearing voice anchor. Conventionally structured by content shape:
  - `writingSamples.howTheyOpenPosts` — array of opener samples (used by `/write-post`)
  - `writingSamples.howTheyClosePosts` — array of closer samples (used by `/write-post`)
  - `writingSamples.signatureRhythm` — array of mid-paragraph rhythm samples
  - `writingSamples.apostropheInContext` — sample(s) showing how the persona uses apostrophes
  - **`writingSamples.replies`** — **(new in v2.1)** flat array of 3 short-form sample replies. Used by `/write-reply` to anchor length and sentence shape. If missing, reply-mode output is best-effort.
- `apostrophes` — informational only. Pattern note (e.g., "mostly keeps, drops in technical product posts"). Not scored against by `/smell`. Drift is expected.
- `casing` — informational only. Same — drift is expected, samples are the source of truth.

### Perspective layer (drives stance, not voice)

- `backstory` — location, age, timeline, struggles, wins, professional context. Surfaces as *which details* the persona references. Does NOT drive sentence-level voice (that's the samples' job).
- `topics` — what the persona writes about. Used for auto-matching and fatigue checks.
- `opinionStrength` — `HARD` / `SOFT` / `EDITORIAL`. Influences how strongly the persona stakes a claim.
- `signaturePhrases` — phrases the persona returns to. Surfaces occasionally in posts, not every time.
- `voice` — a one-line voice description (e.g., "Editorial, strong opinions held loosely. Dual voice: editorial vs product launch."). Informational; the samples are the authoritative voice source.

### Why the split matters

In v2.1, the `/write-*` skills enforce a strict separation:

- **Voice from samples** → sentence shape, casing, slang, punctuation rhythm, length
- **Perspective from backstory** → which specific cafe, which specific job, which specific timeline event

If a persona has rich backstory but no `writingSamples.replies`, `/write-reply` can still generate output but it won't have a length or rhythm anchor. Adding 3 short example replies fixes this and is the recommended setup for any persona that will reply on threads.

### Example: adding reply samples to a persona

```bash
# Via update_persona MCP
update_persona userId=2 personaTraits='{
  "writingSamples": {
    "replies": [
      "yeah same thing happened to me, ended up just deleting the post",
      "the math doesnt add up. their RPM claims are 3x what i see on similar sites",
      "you tried the new export tool yet? it actually works now"
    ]
  }
}'
```

The MCP `update_persona` tool deep-merges `writingSamples`, so this adds the `replies` array without touching the existing `howTheyOpenPosts` / `howTheyClosePosts` / etc.

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
