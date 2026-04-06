# Text Generator (LLM) – Model Selection Knowledge

## Overview

The Text Generator is responsible for transforming natural language prompts into structured or creative text outputs such as scripts, scene descriptions, ad copy, and summaries.

It supports **multimodal context inputs** and outputs **text only**.

---

## Input Modalities

| Modality | Description | Limits |
|---|---|---|
| **Text (required)** | Natural language instructions describing the task or idea | Must fit within model context window |
| **Image (optional)** | Visual references (characters, scenes, products) | JPEG/PNG, ≤10MB |
| **Video (optional)** | Short clips for motion or scene context | ≤90s, ≤50MB |
| **Audio (optional)** | Dialogue or sound references | ≤2min, MP3/WAV |
| **PDF (optional)** | Long-form documents or references | Recommended ≤10MB |

> ⚠️ **Important: modality support differs by model**

- **Gemini 3.1 Pro / Gemini 3.1 Flash**
  - Support: text, image, audio, video（完整多模态）
- **GPT-4o**
  - Support: text + image
  - ❌ 不支持 audio / video 输入

---

## Output Modality

| Modality | Description |
|---|---|
| **Text** | Structured or free-form natural language output |

---

## Confirmed model IDs

Use only the following exact IDs for `textGenerator`. These IDs are confirmed for this atom.

| Display Name | Exact model ID |
|---|---|
| Gemini 3.1 Pro | `google/gemini-3.1-pro-preview` |
| Gemini 3.1 Flash Lite | `google/gemini-3.1-flash-lite-preview` |
| Gemini 3 Flash | `google/gemini-3-flash-preview` |
| Gemini 2.0 Flash | `google/gemini-2.0-flash` |
| GPT 5.2 Pro | `openai/gpt-5.2-pro` |
| GPT 5.2 | `openai/gpt-5.2` |
| GPT 5 | `openai/gpt-5` |
| GPT 4o | `openai/gpt-4o-2024-11-20` |
| GPT 4o Mini | `openai/gpt-4o-mini` |

`scriptSplit` shares the same text-model bucket in practice. Prefer `openai/gpt-5.2` when no stronger instruction is available, and never invent a separate `scriptSplit`-only model ID.

Hard rule:

- Use exact IDs from the table above
- Do not shorten, rename, or upgrade them on your own
- If a prose recommendation is not in the table, do not use it for `selectedModels`

---

## Model Comparison

### Gemini 3.1 Pro

**Capabilities**
- 完整多模态输入：text / image / audio / video
- Context window：最高约 **1M tokens**
- Max output：约 **64K tokens**
- 强推理能力 + 长上下文理解

**Best for**
- 长文本脚本生成（分镜 / 剧本 / Storytelling）
- 多模态输入理解（视频/音频分析）
- 复杂推理 / 多步骤规划

---

### Gemini 3.1 Flash

**Capabilities**
- 同样支持完整多模态输入
- Context window：约 **1M tokens**
- Max output：约 **64K tokens**
- 更快、更低成本

**Best for**
- 大规模批量生成
- 快速改写 / 总结
- 成本敏感场景

---

### GPT-4o

**Capabilities**
- 输入：text + image
- 输出：text
- Context window：约 **128K tokens**
- Max output：约 **16K tokens**
- 指令遵循能力强，结构稳定

**Limitations**
- ❌ 不支持 audio 输入
- ❌ 不支持 video 输入
- 多模态能力主要为图像理解

**Best for**
- 结构化写作（脚本 / outline / 文案）
- 通用文本生成
- 图像理解辅助写作

---

## When to Use Which Model

| Use Case | Recommended Model |
|---|---|
| 多模态输入（视频 / 音频 / 图片） | Gemini 3.1 Pro |
| 高吞吐 + 低成本 | Gemini 3.1 Flash |
| 稳定结构化输出 | GPT-4o |

---

## Common Use Cases

- 脚本生成（分镜 / UGC / 广告）
- 视频结构分析与反推
- 文案创作（广告 / 商品描述）
- 总结与改写
- 为图像 / 视频生成提供结构化输入

---

## Key Limitations

1. **模型多模态能力不同**
   - Gemini 3.1：完整多模态
   - GPT-4o：仅 text + image

2. **上下文差异**
   - Gemini：≈1M tokens
   - GPT-4o：≈128K tokens

3. **提示词结构影响较大**
   - 不结构化输入会显著降低输出质量

4. **PDF处理**
   - 实际按文本抽取处理，不保留完整排版结构

---

## 核心结论

```text
Gemini 3.1 = 多模态理解 + 长上下文 + 复杂推理
GPT-4o = 结构稳定 + 指令遵循强 + 文本任务优先
```
