# Operator Quick-Reference Checklist

This is a compact checklist for Operate Mode. Full details are in `references/api-workflows.md`.

## Before Running

- [ ] Searched templates with 1–3 keywords extracted from user request
- [ ] Merged and deduplicated results across keywords
- [ ] Ranked results by relevance (output type match > keyword match > input type match > model quality)
- [ ] Presented **top 5** candidates in business language with numbered list
- [ ] User selected a template
- [ ] Copied template via `/workflows/from-template` → got `flow_id`
- [ ] Queried `/workflows/{flow_id}/parameters` for real `node_id` list
- [ ] Asked user about every input (text / image / video / audio) — never used `default_value`
- [ ] Translated `node_title` into business language when asking user

## During Running

- [ ] `inputs` payload is flat: `{ "node_id": "string_value" }` — no extra nesting
- [ ] Media URLs are direct file links (not webpage URLs)
- [ ] Local files uploaded to tmpfiles.org (video+image) or catbox.moe (image only)
- [ ] tmpfiles URL converted to `/dl/` direct link

## After Running (强制闭环 — 不可中断)

- [ ] Immediately entered polling loop after run — did NOT stop to wait for user
- [ ] Polling at correct interval (10 s text/image, 30 s video)
- [ ] Reported status to user on each poll ("排队中…" / "生成中…")
- [ ] Continued polling until terminal state (success / failed / cancelled)
- [ ] On success: **immediately** called `/results` and delivered media to user
- [ ] Delivered media directly (not just raw links)
- [ ] Included download link alongside embedded media

## If Something Goes Wrong

1. Task stuck `queued` > 5 min → check `input_overrides` for double nesting
2. Template copy returns edge error → switch to another template
3. All upload services fail → ask user for direct URL
