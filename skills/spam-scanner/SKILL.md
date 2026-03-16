---
name: spam-scanner
description: Scan the production database for spam posts, soft-delete them, and ban spammer accounts. Use when asked to check for spam, clean up spam, or moderate posts.
disable-model-invocation: true
---

# Spam Scanner

Scan the Webmatrices production database for spam posts, delete them, and ban spammer accounts using MCP tools.

## Spam Detection Patterns

Search for these patterns using `search_posts_by_content`:

```
# Indonesian call center / airline / bank scams
hubungi, whatsapp, call center, layanan bantuan, lupa pin,
brimo, blokir, membatalkan, reschedule tiket, restrukturisasi,
costumer service maskapai, cara mengatasi, cara membuka,
terblokir, salah pin, dana bantuan, pinjaman online

# Gambling/adult spam
slot gacor, togel, judi online, obat kuat, vimax, pembesar
```

## Workflow

1. Search for spam using `search_posts_by_content` with key patterns (run multiple searches):
   - `search_posts_by_content` with pattern `hubungi`
   - `search_posts_by_content` with pattern `whatsapp`
   - `search_posts_by_content` with pattern `brimo`
   - `search_posts_by_content` with pattern `blokir`
   - `search_posts_by_content` with pattern `slot gacor`
   - Any other suspicious patterns as needed
2. Also use `list_posts` to review recent posts visually for anything the keyword search missed
3. Display found spam posts with title, author, and user ID
4. **Ask the user for confirmation before taking any action**
5. On confirmation:
   - Use `delete_posts` with the spam post IDs
   - Use `ban_user` for each spammer (with `deleteContent: true` to catch any other posts)
6. Verify with `list_posts` that the spam is gone

## Important

- The MCP tools handle soft-deletion, session cleanup, and audit logging automatically
- Always **ask for confirmation** before deleting or banning
- Show a clear summary of what was found
- Exclude posts by users with emails starting with `bishwasbh` (admin/persona accounts)
