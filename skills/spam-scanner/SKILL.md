---
name: spam-scanner
description: Scan the production database for spam posts, soft-delete them, and ban spammer accounts. Use when asked to check for spam, clean up spam, or moderate posts.
disable-model-invocation: true
---

# Spam Scanner

Scan the Webmatrices production database for spam posts, soft-delete them, and ban the spammer accounts.

## Database Connection

All scripts MUST run from `~/Projects/webm-frontend` for Prisma access. Read production credentials from `.env`:

```bash
cd ~/Projects/webm-frontend
DATABASE_URL=$(grep '^PROD_DATABASE_URL=' .env | cut -d'"' -f2) node temp-script.mjs
```

## Spam Detection Patterns

Match against post `title + body` using these regex patterns:

```
# Indonesian call center / airline / bank scams
/reschedule.*tiket/i
/restrukturisasi/i
/menghubungi\s+cs/i
/layanan\s+bantuan/i
/lupa\s+pin\s+brimo/i
/pembatalan\s+kredit/i
/call\s+center.*24\s+jam/i
/costumer\s+service\s+maskapai/i
/hubungi.*whatsapp.*\+?62/i
/blokir.*brimo/i
/cara\s+(membatalkan|pengajuan|menghubungi)/i

# Phone number spam (Indonesian format)
/\+?62[\-\s]?\d{3,4}[\-\s]?\d{3,4}[\-\s]?\d{2,4}/

# Disguised phone numbers with unicode
/𝐎\d{2,}/
/O899.*call\s+center/i

# Loan/fintech spam
/spinjam/i
/easycash.*whatsapp/i
```

## Workflow

1. Query all posts where `deletedAt IS NULL`
2. Run each post's `title + body` against the spam patterns
3. Display found spam posts with title, author, email, and creation date
4. **Ask the user for confirmation before taking any action**
5. On confirmation:
   - Soft-delete spam posts (`UPDATE posts SET deleted_at = NOW()`)
   - Soft-delete comments on those posts
   - Ban spammer accounts (`UPDATE users SET is_active = false`)
6. Verify the cleanup succeeded
7. Clean up any temporary scripts created

## Important

- Always use **soft-delete** (set `deleted_at`), never hard-delete
- Always **ask for confirmation** before deleting or banning
- Show a clear summary table of what was found
- Clean up temporary `.ts` scripts after execution
- Exclude posts by users with emails starting with `bishwasbh` (admin/persona accounts)
