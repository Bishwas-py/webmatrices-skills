---
name: ban-user
description: Ban a user by ID, username, or email. Sets is_active=false and soft-deletes their posts. Use when asked to ban, deactivate, or block a user.
disable-model-invocation: true
argument-hint: [user-id-or-username-or-email]
---

# Ban User

Ban a user on the Webmatrices platform by setting `is_active = false` and soft-deleting their content.

## Database Connection

All scripts MUST run from `~/Projects/webm-frontend` for Prisma access. Read production credentials from `.env`:

```bash
cd ~/Projects/webm-frontend
DATABASE_URL=$(grep '^PROD_DATABASE_URL=' .env | cut -d'"' -f2) node temp-script.mjs
```

## Workflow

1. Look up the user by the provided identifier:
   - If numeric: search by `id`
   - If contains `@`: search by `email`
   - Otherwise: search by `username`
2. Display the user's details: ID, username, email, post count, comment count, current active status
3. **Ask for confirmation before banning**
4. On confirmation:
   - Set `is_active = false` on the user
   - Soft-delete all their posts (`SET deleted_at = NOW() WHERE deleted_at IS NULL`)
   - Soft-delete all their comments (`SET deleted_at = NOW() WHERE deleted_at IS NULL`)
5. Display confirmation with counts of affected posts/comments
6. Clean up temporary scripts

## Safety

- Never ban users with `is_staff = true` or `is_superuser = true` without explicit confirmation
- Never ban users with emails starting with `bishwasbh` — these are admin accounts
- Always show the user details and ask for confirmation first
- Use soft-delete only, never hard-delete
