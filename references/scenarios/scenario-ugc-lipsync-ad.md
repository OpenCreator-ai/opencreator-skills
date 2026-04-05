# 场景贯穿：UGC 口播广告

> 本文件展示 UGC 口播广告的全链路。提供两条路径：
> - **路径 A（默认优先）**：参考视频仿写 + 音画同步模型直出 — 链路更短、创意可控性更强、支持商品展示
> - **路径 B（仅特定场景）**：产品信息驱动 + TTS + 对口型融合 — 仅在需要严格台词精度或指定音色时使用
>
> **路径选择决策见文件底部。**

---

# 路径 A：参考视频仿写 + 音画同步直出（优先推荐）

> 适用于：用户有参考视频 + 产品图 + 产品信息，想仿写参考视频风格生成 UGC 口播带货视频。
>
> 核心优势：利用 Sora 2 / Kling 3.0 / Seedance 2 等支持音画同步的视频模型，脚本中的台词会被模型直接渲染为口型动作，无需单独的 TTS + lip-sync 融合步骤。

---

## Step 1：结构反推

### 最终产物

UGC 口播带货视频（人物穿着/展示产品，边说话边展示，音画同步）

### 反推过程

```text
口播带货视频
→ 需要：人物关键帧 + 包含台词的动作/场景描述
→ 人物关键帧从哪来？→ 需要 imageToImage（产品图参考 + 人物描述）
→ 人物描述从哪来？→ 需要从脚本中提取人物视觉指令
→ 脚本从哪来？→ 需要仿写 ← 需要参考视频分析
→ 参考视频分析需要什么？→ 参考视频 + 用户需求
→ 产品图从哪来？→ 用户上传
```

### 抽象结构

```text
输入层（参考视频 + 产品图 + 产品信息/需求）
↓
语义层-阶段1（分析参考视频的结构/风格/节奏）
↓
语义层-阶段2（基于分析 + 产品信息 → 仿写新脚本，含台词 + 动作 + 场景）
↓
视觉分支-提取（从脚本提取人物外观描述）
↓
视觉分支-生图（人物描述 + 产品图 → 人物关键帧）
↓
视频生成（人物关键帧 + 脚本描述 → 音画同步口播视频）
↓
输出
```

### 追问检查

- 用户提供了参考视频吗？→ 没有则走路径 B
- 用户提供了产品图吗？→ 没有则追问
- 用户提供了产品信息/URL吗？→ 没有则追问

---

## Step 2：选择 Generator 并连线

| 抽象模块 | Generator | node.type | 节点名 | 说明 |
|----------|-----------|-----------|--------|------|
| 语义层-阶段1 | 参考视频生文器（阶段1） | `textGenerator` | Video Analyzer | 接 videoInput + textInput → 分析参考视频结构/风格 |
| 语义层-阶段2 | 参考视频生文器（阶段2） | `textGenerator` | Script Replicator | 接阶段1 text + imageInput → 仿写脚本（含台词+动作+场景） |
| 视觉分支-提取 | 参考文本生文器 | `textGenerator` | Avatar Creative Instructions | 接阶段2 text + imageInput → 提取人物视觉描述 |
| 视觉分支-生图 | 图片参考生图器 | `imageToImage` | Avatar Key Image | 接人物描述 text + imageInput → 生成人物关键帧 |
| 视频生成 | 图生视频器 | `videoMaker` | Image to Video | 接关键帧 image + 脚本 text → 音画同步直出 |

### 连线图

```text
videoInput ────→ textGenerator A「Video Analyzer」←── textInput
                         │
                         ↓ text（视频结构分析）
imageInput ────→ textGenerator B「Script Replicator」
                         │
                         ↓ text（仿写脚本：台词+动作+场景）
                         │
               ┌─────────┴─────────┐
               ↓                   ↓
imageInput → textGenerator C    （text 直传）
         「Avatar Creative         │
          Instructions」           │
               │                   │
               ↓ text（人物描述）    │
imageInput → imageToImage D        │
         「Avatar Key Image」      │
               │                   │
               ↓ image（关键帧）    ↓ text
               └────→ videoMaker E「Image to Video」
                           │
                           ↓
                         输出（口播视频）
```

### 连线规则校验

- videoInput video → A video ✅
- textInput text → A text ✅
- A text → B text ✅
- imageInput image → B image ✅
- B text → C text ✅
- imageInput image → C image ✅
- C text → D text ✅
- imageInput image → D image ✅
- D image → E image ✅
- B text → E text ✅
- DAG 无环 ✅

### 关键设计说明

- **产品图被复用 3 次**（B、C、D），确保产品一致性贯穿全链路
- **两阶段生文**是参考视频生文器的核心模式：阶段 1 只做分析，阶段 2 才做创作
- **不需要 TTS 和 lip-sync 融合节点**：脚本中的台词通过 text pin 传给 videoMaker，支持音画同步的模型会自动渲染口型

---

## Step 3：模型选择

| 节点 | 推荐模型 | 理由 |
|------|----------|------|
| A textGenerator | `google/gemini-3-flash-lite` | 需要视频理解能力（多模态），Flash Lite 性价比高 |
| B textGenerator | `google/gemini-3-flash-lite` | 需要图片理解 + 上游文本，仿写任务 |
| C textGenerator | `google/gemini-3-flash-lite` | 图片 + 文本 → 提取人物描述 |
| D imageToImage | `gemini-3-pro-image-preview`（Banana Pro）| 写实 UGC 人物，照片级质感，产品一致性 |
| E videoMaker | `fal-ai/sora-2/image-to-video`（Sora 2，**优先**）| 支持音画同步，20s，画质顶级，真人表现力强 |
| E videoMaker 备选 | `fal-ai/kling/v3/pro/image-to-video`（Kling 3.0）| 支持音画同步，中文能力强，稳定性好 |

> **模型选择核心**：E 节点必须选支持音画同步的模型（Sora 2 优先），否则台词不会被渲染为口型动作。
> **注意**：Seedance 2.0 虽支持音画同步，但**不支持真人**，不可用于 UGC 真人场景。

---

## Step 4：提示词

### 缺口审计

| 节点 | 上游已有什么 | 缺什么 | 是否需要写 prompt |
|------|-------------|--------|------------------|
| A | videoInput + textInput | 角色设定 + 分析任务指令 | ✅ |
| B | A 的分析结果 + imageInput | 角色设定 + 仿写任务 + 输出格式 | ✅ |
| C | B 的仿写脚本 + imageInput | 角色设定 + 提取任务 | ✅ |
| D | C 的人物描述 + imageInput | 由上游驱动 | 可选轻量补充（风格控制） |
| E | D 的关键帧 + B 的脚本 | 动作/运镜控制 | 可选轻量补充 |

### 节点 A「Video Analyzer」的 inputText

```text
You are a short-form video analyst.
Input: One reference video.
Analyze and output:
- Video structure (hook → body → CTA breakdown)
- Shot-by-shot description (framing, movement, expression, pace)
- Spoken script / voiceover transcript
- Visual style and tone
- Duration and rhythm pattern
Output in structured format. Be specific and detailed.
```

### 节点 B「Script Replicator」的 inputText

```text
You are an expert in recreating viral short-form video scripts.
Input: A video analysis report + product image.
Task: Write a new UGC video script that replicates the reference video's structure, rhythm, and style, but promotes the product shown in the image.
Output must include:
- Shot-by-shot breakdown (matching reference structure)
- Spoken dialogue/voiceover lines for each shot (口语化、有感染力)
- Character actions, expressions, gestures per shot
- Scene/environment description per shot
Keep total duration ≤20s. Output in structured format. Do not explain.
```

### 节点 C「Avatar Creative Instructions」的 inputText

```text
Extract avatar creative instructions from the given script.
Focus on identifying and summarizing the following aspects:
- Character appearance (age, gender, ethnicity, body type)
- Clothing (must match the product in the reference image)
- Hair, makeup, accessories
- Environment / background setting
- Overall visual style (UGC, realistic, casual)
Output a single cohesive image generation prompt. Do not explain.
```

### 节点 D「Avatar Key Image」的 inputText（可选）

通常上游 C 的描述已足够，如需补充风格控制：

```text
UGC selfie style, natural lighting, iPhone front camera look, casual and authentic
```

### 节点 E「Image to Video」的 inputText（可选）

如需补充动态控制：

```text
[Optional] Describe the character movements, lighting, and visual styles
```

---

## Anti-patterns（本路径高频错误）

- ❌ E 节点选了不支持音画同步的模型 → 台词不会被渲染为口型，变成无声视频
- ❌ B 的脚本没有包含台词/对白 → 视频模型无法生成口播效果
- ❌ 没有用 imageToImage 生成关键帧，直接用 imageMaker → 产品一致性丢失
- ❌ A 和 B 合并成一个节点 → 两阶段模式的分析质量会下降，应分开
- ❌ B 的脚本超过 20 秒 → 超出 videoMaker 单次生成上限
- ❌ 没有把 imageInput 连到 B、C、D → 产品信息断裂，影响一致性

---
---

# 路径 B：产品信息驱动 + TTS + 对口型融合（备选）

> 适用于：用户**没有参考视频**，只有产品图 + 产品信息，想生成 UGC 口播广告视频。
>
> 与路径 A 的区别：无参考视频驱动创意，走传统 TTS + imageAudioToVideo 对口型融合路径。

---

## Step 1：结构反推

### 最终产物

口播视频（人物开口说话，讲产品）

### 反推过程

```text
口播视频
→ 需要：人物画面 + 语音音频
→ 语音从哪来？→ 需要 TTS ← 需要口播脚本 ← 需要产品分析
→ 人物画面从哪来？→ 需要生图 ← 需要图片 prompt ← 需要产品分析
→ 产品分析需要什么？→ 产品图（用户已有）+ 产品信息（用户已有或需追问）
```

### 抽象结构

```text
输入层（产品图 + 产品信息）
↓
语义层（产品分析 → 共享 brief）
↓
├── 视觉分支（图片 prompt → 人物图）
└── 音频分支（口播脚本 → TTS）
↓
合成层（人物图 + 音频 → 对口型视频）
↓
输出
```

### 追问检查

- 用户提供了产品图吗？→ 没有则追问
- 用户提供了产品信息/URL吗？→ 没有则追问
- 用户偏好速度还是质量？→ 决定模型选择

---

## Step 2：选择 Generator 并连线

| 抽象模块 | Generator | node.type | 说明 |
|----------|-----------|-----------|------|
| 语义层 | 参考图生文器 A | `textGenerator` | 接 imageInput + textInput，输出 product_brief |
| 视觉分支-描述 | 参考文本生文器 B | `textGenerator` | 接 A 的 text 输出，生成 ugc_image_prompt |
| 视觉分支-生图 | 文本意图生图器 D | `imageMaker` | 接 B 的 text 输出，生成人物图 |
| 音频分支-脚本 | 参考文本生文器 C | `textGenerator` | 接 A 的 text 输出，生成口播脚本 |
| 音频分支-TTS | 文本转语音 E | `textToSpeech` | 接 C 的 text 输出，生成语音 |
| 合成层 | 图片对口型生视频器 F | `imageAudioToVideo` | 接 D 的 image + E 的 audio，输出成片 |

### 连线图

```text
imageInput ──────────────┐
                         ↓
textInput ───→ textGenerator A（产品分析）
                    │            │
                    ↓            ↓
          textGenerator B    textGenerator C
          （图片prompt）     （口播脚本）
                    │            │
                    ↓            ↓
              imageMaker D   textToSpeech E
                    │            │
                    └────┐  ┌────┘
                         ↓  ↓
                  imageAudioToVideo F
                         │
                         ↓
                       输出
```

### 连线规则校验

- A text → B text ✅
- A text → C text ✅
- imageInput image → A image ✅
- B text → D text ✅
- C text → E text ✅
- D image → F image（上限 1）✅
- E audio → F audio（上限 1）✅
- DAG 无环 ✅

---

## Step 3：模型选择

| 节点 | 推荐模型 | 理由 |
|------|----------|------|
| A textGenerator | `google/gemini-3-pro-preview` | 需要图片理解（多模态） |
| B textGenerator | `openai/gpt-4o-2024-11-20` | 纯文本结构化任务 |
| C textGenerator | `openai/gpt-4o-2024-11-20` | 纯文本脚本生成 |
| D imageMaker | `gemini-3-pro-image-preview`（Banana Pro）| 写实 UGC 人物 |
| E textToSpeech | `fish-audio/speech-1.6` | 风格化语音 |
| F imageAudioToVideo | `fal-ai/infinitalk`（默认） | — |

---

## Step 4：提示词

### 缺口审计

| 节点 | 上游已有什么 | 缺什么 | 是否需要写 prompt |
|------|-------------|--------|------------------|
| A | textInput + imageInput | 角色设定 + 输出格式 | ✅ |
| B | A 的 product_brief | 角色设定 + 输出格式 | ✅ |
| C | A 的 product_brief | 角色设定 + 输出格式 + 时长约束 | ✅ |
| D | B 的 ugc_image_prompt | 由上游驱动 | ❌ |
| E | C 的脚本 | 由上游驱动 | ❌ |
| F | D 的图 + E 的音频 | 可选：表情/风格控制 | 可选轻量补充 |

### 节点 A 的 inputText

```text
你是产品营销分析师。任务是分析输入的产品图片和产品信息，输出结构化的产品 brief。
输出 JSON 格式，包含：product_type、key_features、selling_points、target_users、tone。
不解释、不赘述，直接输出。
```

### 节点 B 的 inputText

```text
你是专业 UGC 广告视觉策划师。任务是基于上游产品 brief，生成一段用于 AI 生图的人物描述。
描述必须包含：主体外貌、环境、动作/表情、景别、机位、构图、灯光、风格。
UGC 风格，真实感，不解释不赘述，直接输出。
```

### 节点 C 的 inputText

```text
你是 UGC 短视频口播脚本撰写师。任务是基于上游产品 brief，生成一段 30 秒以内的口播台词。
语言口语化、有节奏感，适合真人朗读。
不解释不赘述，直接输出台词文本。
```

---

## Anti-patterns（路径 B 高频错误）

- ❌ 没有共享语义层（A），直接让用户写 prompt → 视觉和文案脱节
- ❌ 把 imageAudioToVideo 放在链路中间而非末端
- ❌ C 的脚本超过 30 秒 → TTS 稳定性下降
- ❌ A 节点选了 GPT-4o → 不支持图片输入（需要用 Gemini）
- ❌ D 节点又重复描述主体外貌（B 已经提供了完整描述）→ 造成重复控制

---
---

# 路径选择决策

**默认推荐路径 A**，仅在满足路径 B 全部条件时才走路径 B。

```text
视频中需要展示商品（穿着/手持/使用）？
├── 是 → 路径 A（必选）
│        对口型模型无法很好地参考商品图，路径 A 通过 imageToImage 确保产品一致性
└── 否（纯人像说话）→ 继续判断 ↓

是否必须严格遵循口播台词的每一个字？
├── 是 → 路径 B
│        TTS 保证台词精准朗读，lip-sync 驱动口型
└── 否 → 继续判断 ↓

是否必须使用指定音色（品牌声音 / 特定音色参考）？
├── 是 → 路径 B
│        需要 voiceCloner / textToSpeech 控制音色
└── 否 → 路径 A（优先）
         音画同步模型效果更自然，场景更丰富
```

### 决策总结

| 条件 | 选择 |
|------|------|
| 视频中需展示商品 | **必须路径 A** |
| 有参考视频可仿写 | 路径 A |
| 无特殊音色/台词精度要求 | 路径 A |
| 纯人像 + 必须逐字台词精度 | 路径 B |
| 纯人像 + 必须指定音色 | 路径 B |

> **原则：不轻易用路径 B。** 纯口型驱动模型（imageAudioToVideo）场景受限，只在"严格台词控制"或"指定音色"这两个硬需求同时存在时才推荐。
