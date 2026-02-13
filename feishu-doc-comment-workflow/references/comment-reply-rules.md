# Comment Reply Rules

## Scope

Use these rules when handling Feishu doc comments in mixed ownership scenarios.

## Document classes

1. **Assistant-owned docs**
   - Auto-reply to new comments by default.

2. **Other people's docs**
   - Only auto-reply when latest comment/reply text contains trigger: `【给小八】`.

## Reply behavior

- Keep reply short, friendly, and directly actionable.
- Avoid duplicate replies to the same `comment_id` unless a new user follow-up appears.
- If multiple pending comments exist, process oldest first.

## Trigger matching (customizable)

A comment is eligible when latest user-authored text matches tenant-defined trigger rules.

### Recommended configurable fields

- `externalDocEnabled`: `true|false`
- `matchMode`: `contains|startsWith|regex`
- `triggers`: string list
- `caseSensitive`: `true|false`

### Example profiles

1) **Strict prefix mode**
- `matchMode: startsWith`
- `triggers: ["【给小八】"]`

2) **Multi-trigger contains mode**
- `matchMode: contains`
- `triggers: ["【给小八】", "@小八", "#ask-xiaoba"]`

3) **Regex mode**
- `matchMode: regex`
- `triggers: ["^(\\[?给小八\\]?|@小八)"]`

Default for this skill package: strict prefix `【给小八】`.

## Safety boundaries

- Do not post to unrelated chats from comment events.
- Do not expose private data from other docs.
- If doc cannot be resolved (wiki/docx token mismatch), fail closed and ask for mapping or permissions.
