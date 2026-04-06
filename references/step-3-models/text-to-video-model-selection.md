# Text to Video – 模型选型知识

## 概述

Text to Video 用于将自然语言描述生成视频内容。

用户输入一段创意指令，描述故事、主体、动作、视觉风格，以及可选的音频信息，模型输出一段完整视频。

本模块只关注：
- 输入 / 输出
- 模型如何选
- 每个模型支持哪些高级参数
- 时长如何拆
- 比例如何选

---

## 输入（Input）

| 模态 | 说明 | 限制 |
|---|---|---|
| **Text（必选）** | 视频内容描述，包括主体、动作、风格、场景等 | 建议简洁、结构化 |

---

## 输出（Output）

| 模态 | 说明 | 用途 |
|---|---|---|
| **Video** | 生成的视频片段，可包含动作、构图，以及可选音频 | 广告、叙事、社媒内容、产品展示 |

---

## Confirmed model IDs

只使用下面这些已确认归属于 `textToVideo` 的模型 ID。

| 展示名 | 精确 model id |
|---|---|
| Sora 2 | `fal-ai/sora-2/text-to-video` |
| Sora 2 Pro | `fal-ai/sora-2/text-to-video/pro` |
| Veo 3.1 Fast | `veo-3.1-fast-generate-preview` |
| Veo 3.1 | `veo-3.1-generate-preview` |
| Veo 3 Fast | `fal-ai/veo3/fast` |
| Veo 3 | `fal-ai/veo3` |
| Kling o3 Pro | `fal-ai/kling-video/o3/pro/text-to-video` |
| Kling 3.0 Pro | `fal-ai/kling-video/v3/pro/text-to-video` |
| Kling 3.0 Standard | `fal-ai/kling-video/v3/standard/text-to-video` |
| Kling 2.6 Pro | `fal-ai/kling-video/v2.6/pro/text-to-video` |
| Seedance 2.0 | `doubao-seedance-2-0-260128` |
| Seedance 2.0 Fast | `doubao-seedance-2-0-fast-260128` |
| Seedance 1.5 Pro | `fal-ai/bytedance/seedance/v1.5/pro/text-to-video` |
| Seedance 1.0 Lite | `fal-ai/bytedance/seedance/v1/lite/text-to-video` |
| Wan 2.6 | `wan/v2.6/text-to-video` |
| Vidu Q3 | `fal-ai/vidu/q3/text-to-video` |
| Hailuo Video 02 | `fal-ai/minimax/hailuo-02/standard/text-to-video` |
| Grok Imagine Video | `xai/grok-imagine-video/text-to-video` |

硬规则：

- `textToVideo` 只能使用上表里明确映射到 `textToVideo` 的 id
- 不要把 image-to-video 的 `/image-to-video` 或 `/i2v` id 填到 `textToVideo`
- 推荐名不是配置值，`selectedModels` 必须写精确 id

---

## 模型选型

本模块支持多种视频生成模型，不同模型在场景表现、参数能力和稳定性上存在差异。

### Sora 2
- 电影级质量与动作真实感强
- 支持**音画同步**
- 适合高质量叙事、广告、单镜头高完成度生成
- 在真实使用中，**UGC / 广告类场景优先级最高**

### Veo 3.1 Fast
- 速度快、稳定性好
- 支持**音画同步**
- 适合快速试错、批量生成、短视频迭代

### Kling 3.0 Standard / Kling 2.6 Pro
- 中文语境表现强
- 时长灵活、控制能力强
- 支持音频能力（Standard 支持 audio_mode）
- 综合表现仅次于 Sora
- 特别适合中文 UGC、广告批量生产

### Seedance 2.0
- 视觉一致性强、风格化能力强
- 对结构化输入表现好
- 支持**音画同步**
- 更适合可控、稳定的非真人内容
- **限制：暂时不支持真人**

---

## 实际优先级（UGC / 广告）

```text
Sora 2 > Kling 3.0 > Seedance 2.0 > Veo 3.1 Fast
```

### 关键约束

```text
真人出镜 / 真人露出 → 不可使用 Seedance 2.0
需要最高综合质量 → 优先 Sora 2
中文 UGC / 广告批量生产 → 优先 Kling
快速试错 / 批量变体 → 优先 Veo 3.1 Fast
```

---

## 能力对比

| 能力 | 支持模型 |
|---|---|
| **音画同步** | Sora 2, Veo 3.1, Kling（Standard 开启音频时）, Seedance 2.0 |
| **中文 prompt 表现** | Kling 系列, Seedance |
| **快速生成** | Veo 3.1 Fast |
| **电影级质量** | Sora 2 |

---

## 参数

### 通用参数

| 参数 | 选项 | 说明 |
|---|---|---|
| `aspect_ratio` | `16:9`, `9:16`, `1:1` 等 | 控制视频画幅与布局 |
| `duration` | 如 `4s`, `5s`, `8s`, `12s`, `15s`, `20s` | 控制生成视频时长 |

---

### 模型专属参数

#### Veo 3.1 Fast

| 参数 | 选项 | 说明 |
|---|---|---|
| `resolution` | `720p`, `1080p` | 输出视频质量 |
| `duration` | `4s`, `6s`, `8s` | 适合短视频生成 |

---

#### Sora 2

| 参数 | 选项 | 说明 |
|---|---|---|
| `aspect_ratio` | `16:9`, `9:16` | 输出画幅 |
| `duration` | `8s`, `12s`, `16s`, `20s` | 视频时长 |

---

#### Kling 3.0 Standard / 2.6 Pro

| 参数 | 选项 | 说明 |
|---|---|---|
| `aspect_ratio` | `16:9`, `9:16`, `1:1` | 视频布局 |
| `audio_mode` | `No Native Audio`, `Native Audio`, `Voice Control` | 控制音频生成方式 |
| `duration` | `3s – 15s`（Standard），Pro 为固定档位 | 视频时长 |

---

#### Seedance 2.0

| 参数 | 选项 | 说明 |
|---|---|---|
| `aspect_ratio` | `16:9`, `4:3`, `1:1`, `9:16`, `3:4`, `21:9` | 视频画幅 |
| `resolution` | `480p`, `720p` | 输出清晰度 |
| `duration` | `4s – 14s` | 视频时长 |
| `generate_audio` | `on`, `off` | 是否生成音频 |

---

## 时长选择（新增）

### 核心原则

```text
单个视频素材时长 = 总需求时长 ÷ 镜头数
```

### 推导方法

例如：

```text
总需求时长 30s
→ 5 个镜头
→ 每个片段约 6s
```

### 推荐区间

```text
4s–6s  → 最稳，推荐默认
6s–8s  → 常用区间
8s–12s → 适合 Sora 2 / Seedance 2.0 / 部分 Kling
12s–20s → 仅建议优先使用更强模型（Sora 2），风险明显上升
```

### 结论

```text
优先拆成多个短镜头素材，再做拼接
而不是一次生成长视频
```

---

## 比例选择（新增）

### 核心原则

```text
比例由最终使用场景倒推
```

### 常见映射

```text
9:16 → TikTok / Reels / Shorts / 竖版广告
4:5  → Feed 广告（若平台支持该视频比例）
16:9 → 横版内容 / YouTube / 官网展示
1:1  → 通用 / 中性 / 部分社媒
```

### 与模型参数的关系

不同模型虽然都在控制画幅，但可选范围不同：

- **Sora 2** → `aspect_ratio`: `16:9`, `9:16`
- **Veo 3.1 Fast** → 使用通用 `aspect_ratio`
- **Kling 3.0 / 2.6** → `aspect_ratio`: `16:9`, `9:16`, `1:1`
- **Seedance 2.0** → `aspect_ratio`: `16:9`, `4:3`, `1:1`, `9:16`, `3:4`, `21:9`

### 实际选择建议

```text
竖版广告优先：9:16
横版叙事优先：16:9
通用测试优先：1:1
需要更强海报感 / 广告感：可考虑 4:5 / 3:4（若模型支持）
```

---

## 常见用途

- **UGC 广告生成**  
  具有自然动作的短营销视频

- **Storytelling / 叙事短片**  
  从文本生成电影感视频片段

- **产品视频**  
  动态展示产品使用场景

- **社交媒体内容**  
  TikTok / Reels 风格竖版视频

- **创意预演**  
  在正式制作前快速可视化想法

---

## 限制

- 动作一致性会受 prompt 复杂度影响
- 音频质量依赖模型及音频模式
- 时长越长，视觉稳定性通常越弱
- 大多数模型更适合**单镜头片段**，多镜头需要分段生成再拼接

---

## 核心结论

```text
Sora / Kling = UGC 与广告主力模型
Seedance 2.0 = 非真人、稳定、可控
Veo = 快速试错与高吞吐

时长靠总时长倒推到单镜头
比例靠最终使用场景来选
模型决定能力，参数决定控制边界
```
