# OpenCreator Operator Playbook

Use this file for the parts that are not just "call an API". It covers how to search templates well, how to ask the user for missing inputs, how to prepare media URLs, how to diagnose queued runs, and how to return outputs cleanly.

## 1. Template Search Workflow

Follow this sequence:

1. Extract 1 to 3 keywords from the user's request
2. Search templates
3. Merge results across keywords
4. Deduplicate by `template_id`
5. Show the best few candidates in user language

### Keyword Extraction Rules

- Prefer concrete nouns and action words from the user's original request
- Match the user's language first
- If Chinese keywords do not return good matches, retry with English equivalents
- Avoid broad filler words such as "生成", "workflow", or "AI" unless the request is otherwise too sparse

Examples:

- "帮我做珠宝亚马逊上架图" -> `珠宝`, `亚马逊`
- "I want viral UGC video" -> `viral`, `UGC`
- "帮我找 AI 生视频的" -> `视频`

### How To Present Templates

Show templates in user-facing format:

```text
模板名称
一句话描述
模型：xxx、xxx
```

Do not expose internal IDs until you need to act on the chosen template.

## 2. Input Collection Rules

After calling the parameters API, collect inputs from the user carefully.

### Non-Negotiable Rules

- Ask about every returned input node
- Do not use `default_value` automatically
- If the user already provided a value, reuse it and only ask about missing fields
- Translate technical fields into business language

### How To Map Input Types

- `text`
  - Ask for the text content the node needs
- `image`
  - Ask for an image file or a direct image URL
- `video`
  - Ask for a video file or a direct video URL
- `audio`
  - Ask for an audio file or a direct audio URL

### Face-User Naming Rules

Never say:

- `node_id`
- `node_type`
- `field_key`
- `inputText`
- `imageBase64`

Instead, use the node title's business meaning:

- `Text Input` -> "请描述你的产品"
- `Image Input` -> "请上传你的商品图"
- `Reference Video` -> "请发给我你想复刻风格的参考视频"

## 3. Run Payload Rules

The run API is easy to get subtly wrong.

### Correct Shape

`inputs` must be a flat map:

```json
{
  "inputs": {
    "textInput-abc123": "你的文本",
    "imageInput-def456": "https://example.com/image.png"
  }
}
```

### Wrong Shape

Do not wrap values again:

```json
{
  "inputs": {
    "textInput-abc123": {
      "inputText": "你的文本"
    }
  }
}
```

This can leave the task stuck in `queued`.

## 4. Polling And Diagnosis

Use different polling intervals based on likely task duration:

- Text and image workflows: poll every 10 seconds
- Video or multi-step heavy workflows: poll every 30 seconds

### Queued Diagnosis

If a task stays `queued` for more than 5 minutes:

1. Inspect the run status payload
2. Look at `input_overrides`
3. Check for accidental double nesting such as:

```json
{
  "inputText": {
    "inputText": "..."
  }
}
```

If you see this, flatten the `inputs` payload and rerun.

## 5. Media URL Preparation

OpenCreator inputs expect direct URLs for remote media.

### Preferred Upload Services

1. `tmpfiles.org`
   - Use for images and videos
2. `catbox.moe`
   - Use for images only
3. Ask the user for a direct link
   - Use when upload services fail

### tmpfiles Rule

Convert tmpfiles share URLs to download URLs:

- Share URL:
  - `https://tmpfiles.org/12345/file.mp4`
- Direct URL:
  - `https://tmpfiles.org/dl/12345/file.mp4`

### catbox Rule

- Use only for images
- Do not use for videos

## 6. Result Delivery Rules

When a run succeeds:

- `image`
  - Return the image itself if the client supports image display
- `video`
  - Return the video itself if the client supports video display
- `audio`
  - Return the audio itself if the client supports audio playback
- `text`
  - Return the text directly in the message

Also include the raw direct link so the user can download the asset.

Do not say only "生成成功，链接如下" unless the client cannot render media.

## 7. Hard Do-Nots

- Do not run a public template directly without copying it first
- Do not skip the parameters API before a run
- Do not expose technical field names to the user
- Do not assume the template's default values match the user's intent
- Do not send webpage URLs where a direct media file URL is required
- Do not call the results API before run status is `success`
- Do not use `catbox.moe` for video uploads
