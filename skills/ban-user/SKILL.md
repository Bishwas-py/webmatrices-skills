---
name: ban-user
description: Ban a user by ID, username, or email. Sets is_active=false and soft-deletes their posts. Use when asked to ban, deactivate, or block a user.
disable-model-invocation: true
argument-hint: [user-id-or-username-or-email]
---

# Ban User

Ban a user on the Webmatrices platform using the MCP tools.

## Workflow

1. Look up the user by the provided identifier:
   - If numeric: use `get_user_details` with `userId`
   - If contains `@`: use `list_users` with `search` (the email)
   - Otherwise: use `get_user_details` with `username`
2. Display the user's details: ID, username, email, post count, comment count, current active status
3. **Ask for confirmation before banning**
4. On confirmation: use `ban_user` with `userId`, `deleteContent: true`, and a `reason`
5. Display the result (affected post/comment counts)

## Safety

- Never ban users with `is_staff = true` or `is_superuser = true` without explicit confirmation
- Never ban users with emails starting with `bishwasbh` — these are admin accounts
- Always show the user details and ask for confirmation first
- The MCP `ban_user` tool handles: deactivation, content soft-deletion, session cleanup, and audit logging
