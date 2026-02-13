# Troubleshooting (Feishu Doc Comment Workflow)

## 1) `list_comments` returns 404 `code:1069307 not exist`

Likely causes:
- Wiki token was used directly as doc token.
- App lacks wiki permission to resolve wiki -> obj_token.

Fix:
1. Run `feishu_wiki.get` with wiki token.
2. Use returned `obj_token` (docx) for `feishu_doc.list_comments`.
3. Ensure wiki scope exists: `wiki:wiki` or `wiki:wiki:readonly`.

---

## 2) Can read doc but cannot read/reply comments

Likely cause:
- Missing comment scopes.

Required scopes:
- `docs:document.comment:read`
- `docs:document.comment:create` (or write_only equivalent)

---

## 3) `write/append` returns 400 on old doc token

Likely causes:
- Legacy/invalid token
- Permission mismatch

Fix:
1. Create a fresh doc with `feishu_doc.create`.
2. Retry write/append on new token.

---

## 4) Webhook mode configured but no events handled

Likely cause:
- Runtime monitor webhook branch missing/disabled.

Fix:
1. Ensure webhook listener is implemented and enabled.
2. Verify `connectionMode=webhook`, `webhookPort`, `webhookPath`.
3. Match callback URL in Feishu Open Platform.
4. Restart gateway after config/patch changes.

---

## 5) Immediate response required but realtime unavailable

Use polling fallback:
- 3s~12s cadence depending on stability target.
- Route rules:
  - assistant-owned docs: reply all new comments
  - external docs: reply by configurable trigger rules (not hardcoded)

---

## 6) Duplicate append / repeated write for one comment

Likely causes:
- Same `comment_id` reprocessed by concurrent polling runs.
- Missing idempotency check before write.

Fix:
1. Before write, check whether same `comment_id` already has assistant completion receipt.
2. Re-read doc and verify target text does not already exist.
3. If exists, skip write and return `已存在，未重复追加`.

---

## 7) Ordered list sequence is wrong

Likely cause:
- Generated content not normalized before append.

Fix:
1. Normalize ordered list numbering to strict `1..N` before writing.
2. If order cannot be guaranteed, ask user for clarification instead of writing malformed list.
