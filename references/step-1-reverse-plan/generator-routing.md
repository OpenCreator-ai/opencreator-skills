# Generator 概念 → node.type 映射与选择决策表

本文件是 Step 2（选择 Generator）的核心查找表。在完成 Step 1 结构反推后、选择具体 Generator 之前，**必须查阅本文件**。

---

## 一、映射表

### 文本生成类

| Generator 概念名 | node.type | 输入 Pin | 输出 Pin | 核心用途 | 对应 Skill 文件 |
|---|---|---|---|---|---|
| 参考文本生文器 | `textGenerator` | text | text | 纯文本输入 → 结构化脚本 | `reference-text-generator.md` |
| 参考图生文器 | `textGenerator` | text, image | text | 图片 → 语义提取 / 卖点分析 / 脚本 | `reference-image-text-generator.md` |
| 参考视频生文器 | `textGenerator` | text, video | text | 视频 → 结构拆解 → 脚本仿写（两阶段） | `reference-video-text-generator.md` |
| 多模参考生文器 | `textGenerator` | text, image, video, audio | text | 多源融合 → 统一结构化脚本 | `multimodal-text-generator.md` |
| 故事板分文器 | `scriptSplit` | text | text (list) | 结构化脚本 → 文本列表（触发列表态） | `storyboard-text-splitter.md` |

> **注意**：以上四种"生文器"在 node.type 层面都是 `textGenerator`，区别在于**连接了哪些输入 Pin** 和 **inputText 中的角色设定**。选择哪种生文器取决于用户提供了哪些模态的素材。

### 图像生成类

| Generator 概念名 | node.type | 输入 Pin | 输出 Pin | 核心用途 | 结构模式 |
|---|---|---|---|---|---|
| 文本意图生图器 | `imageMaker` | text | image | 纯文本 → 图片 | 单次 |
| 图片参考生图器 | `imageToImage` | image, text | image | 参考图 + 文本 → 融合图片 | 单次 |
| 故事板分镜生图器 | `imageToImage` | image, text | image (list) | 1 张 Input Image × N 段文本 → N 张分镜图 | **广播** |
| 打光生图器 | `relight` | image | image | 光照后处理（不改内容） | 后处理 |
| 换角度生图器 | `imageAngleControl` | image | image | 视角后处理（不改内容） | 后处理 |

> **故事板分镜生图器**的关键约束：Text 必须来自 `scriptSplit` 列表态输出，Image 必须来自 `imageInput`（不可使用 Generated Image，否则退化为单图输出）。

### 视频生成类

| Generator 概念名 | node.type | 输入 Pin | 输出 Pin | 核心用途 | 结构模式 |
|---|---|---|---|---|---|
| 文生视频器 | `textToVideo` | text | video | 纯文本 → 单段视频（无图片时使用） | 单次 |
| 图生视频器 | `videoMaker` | image, text | video | 单图 → 单段视频 | 单次 |
| 参考图分镜生视频器 | `videoMaker` | image, text | video (list) | 1 张 Input Image × N 段文本 → N 段视频 | **广播** |
| 故事板分镜生视频器 | `videoMaker` | image, text | video (list) | N 张分镜图 × N 段文本 → N 段视频 | **编号对齐** |
| 多模入参生视频器 | `omniVideoGeneration` | text(必填), image(≤9), video(≤3), audio(≤3) | video | 多模态参考 → 单段综合短视频（≤20s） | 单次 |
| 图片对口型生视频器 | `imageAudioToVideo` | image, audio, text | video | 人像图 + 语音 → 对口型说话视频 | 单次 |
| 动作模仿生视频器 | `klingMotionControl` | image, video, text | video | 主体图 + 动作参考视频 → 动作迁移视频 | 单次 |
| 视频修改生视频器 | `videoToVideo` | video, text, subject, style | video | 已有视频 → 修改 / 风格迁移 / 主体替换 | 单次 |

> **参考图分镜生视频器**与**故事板分镜生视频器**都映射到 `videoMaker`，区别在于输入结构：
> - 广播：Image 来自 `imageInput`（单张），Text 来自 `scriptSplit`（列表）
> - 编号对齐：Image 来自上游生成的分镜图列表，Text 来自 `scriptSplit`（列表），两者数量必须一致

### 音频生成类

| Generator 概念名 | node.type | 输入 Pin | 输出 Pin | 核心用途 |
|---|---|---|---|---|
| 文本转语音生音频器 | `textToSpeech` | text | audio | 文本 → 配音语音 |
| 音色转语音生音频器 | `voiceCloner` | audio, text | audio | 音色参考 + 文本 → 克隆音色语音 |
| 音乐生音频器 | `musicGenerator` | text | audio | 风格描述 → 背景音乐 |

---

## 二、Generator 选择决策表

完成 Step 1 结构反推后，按最终产物类型查表选择 Generator。

### 2.1 需要生成文本 / 脚本时

```text
用户提供了什么素材？
├── 只有文本 → 参考文本生文器 (textGenerator)
├── 有图片（产品图 / 参考图） → 参考图生文器 (textGenerator + image 输入)
├── 有参考视频 → 参考视频生文器 (textGenerator + video 输入，两阶段)
└── 多种模态混合 → 多模参考生文器 (textGenerator + 多模态输入)

需要批量分镜 / 多图 / 列表态？
├── 是 → 上游生文器后接 故事板分文器 (scriptSplit)
└── 否 → 直接输出结构化脚本，不接 scriptSplit
```

### 2.2 需要生成图片时

```text
有参考图片？
├── 有
│   ├── 需要批量分镜图（上游有 scriptSplit 列表态）？
│   │   ├── 是 → 故事板分镜生图器 (imageToImage，广播模式)
│   │   │       Image 必须来自 imageInput，Text 来自 scriptSplit
│   │   └── 否 → 图片参考生图器 (imageToImage，单次)
│   └── 需要后处理？
│       ├── 改光照 → 打光生图器 (relight)
│       └── 改视角 → 换角度生图器 (imageAngleControl)
└── 无参考图
    └── 文本意图生图器 (imageMaker)
```

### 2.3 需要生成视频时

```text
已有视频需要修改？
├── 是 → 视频修改生视频器 (videoToVideo)
└── 否 → 继续判断 ↓

需要动作迁移（主体图 + 动作参考视频）？
├── 是 → 动作模仿生视频器 (klingMotionControl)
└── 否 → 继续判断 ↓

多模态快速综合生成（≤20s，不需精细分镜）？
├── 是 → 多模入参生视频器 (omniVideoGeneration)
└── 否 → 继续判断 ↓

需要口播 / 说话？
├── 是 → 视频中需要展示商品（穿着/手持/使用）？
│   ├── 是 → 音画同步 videoMaker 直出（必选路径 A）
│   │        脚本含台词+动作+场景，videoMaker 选 Sora 2 / Kling 3.0 等音画同步模型
│   │        对口型模型无法很好地参考商品图
│   └── 否（纯人像说话）→ 继续判断 ↓
│       是否必须严格逐字遵循口播台词 或 必须指定音色？
│       ├── 是 → 图片对口型生视频器 (imageAudioToVideo，路径 B)
│       │        需要上游 TTS / voiceCloner 提供音频
│       └── 否 → 音画同步 videoMaker 直出（优先路径 A）
│                不轻易用路径 B，纯口型驱动模型场景受限
└── 否（不需要口播）→ 继续判断 ↓

有图片作为首帧？
├── 有
│   ├── 上游有 scriptSplit 列表态？
│   │   ├── 是 → 图片也是列表态（来自上游生成的分镜图列表）？
│   │   │   ├── 是 → 故事板分镜生视频器 (videoMaker，编号对齐)
│   │   │   │       Image List 数量必须 = Text List 数量
│   │   │   └── 否（单张 Input Image） → 参考图分镜生视频器 (videoMaker，广播)
│   │   │       Image 必须来自 imageInput
│   │   └── 否 → 图生视频器 (videoMaker，单次)
│   └── 否 → 图生视频器 (videoMaker，单次)
└── 无图片
    └── 文生视频 (textToVideo)
```

> **口播视频路径选择原则**：默认走音画同步 videoMaker 直出（路径 A），仅在"严格逐字台词精度"或"指定音色"硬需求时才走 TTS + imageAudioToVideo（路径 B）。详见 `scenarios/scenario-ugc-lipsync-ad.md` 底部的完整决策树。

### 2.4 需要生成音频时

```text
需要语音还是音乐？
├── 语音
│   ├── 有音色参考？
│   │   ├── 是 → 音色转语音生音频器 (voiceCloner)
│   │   └── 否 → 文本转语音生音频器 (textToSpeech)
│   └── 文本较长？
│       └── 建议先 scriptSplit 拆分，再并行生成
└── 音乐
    └── 音乐生音频器 (musicGenerator)
```

---

## 三、常见结构模式速查

### 模式 A：单链路（最简单）

```text
输入 → 生成器 → 输出
```

适用：文生图、文生视频、图生视频等单次生成。

### 模式 B：参考视频仿写 + 音画同步直出（UGC 优先模式）

```text
参考视频 + 产品图 + 需求
    → 生文器 A（视频分析）→ 生文器 B（脚本仿写，含台词+动作+场景）
                                   │
                        ┌──────────┴──────────┐
                        ↓                     ↓
                  生文器 C（人物描述）    （脚本 text 直传）
                        ↓                     │
    产品图参考 → imageToImage D（关键帧）     │
                        │                     │
                        └────→ videoMaker E ←──┘
                          （音画同步直出，Sora 2 优先）
                                   ↓
                                 输出
```

适用：UGC 口播带货、展示商品的说话视频。关键：脚本包含台词，videoMaker 选音画同步模型直接渲染口型，无需 TTS + lip-sync。产品图贯穿多节点保证一致性。

### 模式 C：共享语义层 + 双分支 + lip-sync 融合

```text
输入 → 生文器 A（语义分析）
         ├── 生文器 B（视觉描述） → 生图器 D
         └── 生文器 C（口播脚本） → TTS E
                                        ↓
                              D + E → 对口型 F → 输出
```

适用：纯人像说话 + 必须严格台词精度或指定音色。关键：A 是共享语义层，保证视觉与文案同源。注意：不轻易使用，仅在硬需求时选择。

### 模式 D：分镜列表态（广播）

```text
输入 → 生文器 → scriptSplit → [Text List]
                                    +
                              imageInput
                                    ↓
                              生图器（广播：1图 × N文本 → N图）
                                    ↓
                              [Image List]
```

适用：多图、分镜图生成。

### 模式 E：分镜列表态（编号对齐）

```text
[Text List]  +  [Image List]
         ↓           ↓
    故事板分镜生视频器（Text #i + Image #i → Video #i）
                    ↓
              [Video List]
```

适用：长视频分镜，每个镜头有独立参考图。前提：Text List 与 Image List 数量一致。
