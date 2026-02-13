---
name: feishu-doc-comment-workflow
description: "End-to-end Feishu doc collaboration workflow: create/write docs, read/reply comments, and execute comment-driven doc edits for both assistant-owned and external docs with configurable trigger rules, idempotent safeguards, and permission troubleshooting. Use for Feishu/Lark docx/wiki collaboration and auto comment workflows."
---

# Feishu Doc Comment Workflow

## Overview

Use this skill for practical Feishu document collaboration: create document, write content, read comments, reply comments, and quickly diagnose permission or token-type failures.

## Quick workflow

1. **Identify token type**
   - `.../docx/<token>` → use `feishu_doc` directly.
   - `.../wiki/<token>` → try `feishu_wiki.get` to resolve `obj_token`; if unavailable, ask user for the docx link once and persist it.

2. **Create + write content**
   - `feishu_doc { action: "create", title }`
   - `feishu_doc { action: "write" | "append", doc_token, content }`

3. **Comment loop**
   - `feishu_doc { action: "list_comments", doc_token, include_replies: true }`
   - For each target comment: `feishu_doc { action: "reply_comment", doc_token, comment_id, content }`

4. **Verification**
   - Re-run `list_comments` to confirm new `reply_id` appears.

## Trigger rules for comment auto-reply

When user sets explicit policy, apply these defaults:

- Docs created/owned by assistant workflow: auto-reply all new comments.
- Other docs: apply configurable trigger rules (default example may use `【给小八】`, but should be tenant-configurable).
- If comment matches configurable edit trigger + edit verb (see `references/comment-edit-rules.md`), execute edit flow then post change receipt in thread.
- Enforce idempotency: same `comment_id` must not cause duplicate writes.
- Enforce minimal-diff mode: do not overwrite whole document unless explicitly confirmed.
- If realtime trigger is not available, use polling fallback.

## Failure patterns and fixes

### A) `list_comments` returns 400 with code `99991672`
Missing scopes. Ask user to grant one of the required doc/comment scopes shown by the API error.

Commonly needed scopes:
- `docs:document.comment:read`
- `docs:document.comment:create` (for replying)
- plus one doc/drive readable scope family (`docs:doc:readonly` / `drive:drive:readonly`, etc.) depending on tenant setup.

### B) `list_comments` returns 404 `not exist`
Usually token mismatch (wiki token used as doc token) or cross-space visibility issue.

Fix order:
1. Confirm link type (`/wiki/` vs `/docx/`).
2. Use resolved `docx` token.
3. Re-test with `feishu_doc.read` and then `list_comments`.

### C) Can read doc but cannot list comments
This is normal when basic doc scopes exist but comment scopes are missing.
Request comment-specific scopes.

## Minimal command templates

```json
{"action":"create","title":"文档标题"}
```

```json
{"action":"write","doc_token":"<doc_token>","content":"# 标题\n正文"}
```

```json
{"action":"list_comments","doc_token":"<doc_token>","include_replies":true}
```

```json
{"action":"reply_comment","doc_token":"<doc_token>","comment_id":"<comment_id>","content":"收到，已处理。"}
```

## References

- `references/comment-reply-rules.md`  
  Load when setting or explaining auto-reply policy across assistant-owned docs vs other people's docs.
- `references/comment-edit-rules.md`  
  Load when user asks to edit document content directly via comments (instruction parsing, guardrails, change receipt).
- `references/troubleshooting.md`  
  Load when encountering 400/404, permission gaps, wiki/docx mapping issues, or webhook/polling fallback decisions.

## Output style

- User-facing updates should be short and action-oriented.
- When blocked, provide exactly:
  1) what failed,
  2) one required permission/link fix,
  3) immediate next step.
