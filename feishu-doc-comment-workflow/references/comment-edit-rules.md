# Comment-Driven Doc Edit Rules (v2)

## Goal

Allow users to request document edits directly from comments, auto-apply **minimal changes only**, and reply with a clear change receipt.

## Trigger format (configurable)

Do not hardcode `【给小八】`.

Use configurable trigger settings, for example:
- `editTriggerEnabled`: `true|false`
- `triggerMatchMode`: `contains|startsWith|regex`
- `triggerPrefixes`: string list (e.g. `["【给小八】", "@DocBot", "#doc"]`)
- `editVerbs`: string list (e.g. `["修改", "改文档", "重写"]`)

A comment is treated as edit command only when:
1) it matches configured prefix rule, and
2) it contains one configured edit verb.

If not matched, treat as normal Q&A reply.

## Default execution policy

- Prefer **append** or **local/small rewrite**.
- Do **not** perform full-document overwrite by default.
- Only change the part explicitly requested by user.

## Idempotency (must)

Before any write:
1. Check whether the same `comment_id` already has assistant completion receipt (`已完成文档修改`). If yes, skip writing.
2. Re-read document and detect whether target text already exists. If yes, skip duplicate write and reply `已存在，未重复追加`.

## Supported edit intents

1. **Append content**
- Example: `【给小八】修改：在文末新增“下一步计划：...”`
- Action: `feishu_doc.append`

2. **Scoped rewrite (small area only)**
- Example: `【给小八】重写：把第二段改得更口语化，保留原意`
- Action:
  - read current content
  - rewrite only requested scope
  - write back only if platform supports safe scoped update; otherwise require confirmation.

## Formatting quality rules

- Ordered lists must be strictly ascending `1..N`.
- Keep unaffected sections unchanged.
- If request is ambiguous, ask a clarification question in comment thread instead of guessing.

## Guardrails

Require confirmation (do not execute immediately) when command implies high risk:
- delete whole section / clear document / major structural rewrite
- legal/financial critical wording changes without explicit instruction

Confirmation phrase:
- `【给小八】确认执行`

## Receipt template

After processing, always reply in the same comment thread:
- `已完成文档修改 ✅`
- `变更类型：append / scoped-rewrite / skipped`
- `变更摘要：...`
