# Image to Video – 模型选型知识

## 概述

Image to Video 用于将**静态图片转化为动态视频**。

核心特点：
- 图片作为视觉锚点（主体 / 构图 / 风格已确定）
- 文本仅用于补充动作、运镜、情绪等动态信息

本模块只关注：
- 输入 / 输出
- 模型如何选
- 时长如何拆
- 比例如何选

---

## 输入（Input）

| 模态 | 说明 |
|---|---|
| **Image（必选）** | 作为主体与构图参考 |
| **Text（可选）** | 补充动作 / 运镜 / 情绪 |

---

## 输出（Output）

| 模态 | 说明 |
|---|---|
| **Video** | 基于图片生成的视频 |

---

## Confirmed model IDs

只使用下面这些已确认归属于 `videoMaker` 的模型 ID。

| 展示名 | 精确 model id |
|---|---|
| Sora 2 | `fal-ai/sora-2/image-to-video` |
| Sora 2 Pro | `fal-ai/sora-2/image-to-video/pro` |
| Veo 3.1 Fast | `veo-3.1-fast-generate-preview/i2v` |
| Veo 3.1 | `veo-3.1-generate-preview/i2v` |
| Veo 3 Fast | `fal-ai/veo3/fast/image-to-video` |
| Veo 3 | `fal-ai/veo3/image-to-video` |
| Veo 2 | `fal-ai/veo2/image-to-video` |
| Kling 3.0 Standard | `fal-ai/kling-video/v3/standard/image-to-video` |
| Kling 3.0 Pro | `fal-ai/kling-video/v3/pro/image-to-video` |
| Kling o3 Standard | `fal-ai/kling-video/o3/standard/image-to-video` |
| Kling o3 Pro | `fal-ai/kling-video/o3/pro/image-to-video` |
| Kling 2.6 Pro | `fal-ai/kling-video/v2.6/pro/image-to-video` |
| Kling 2.5 Pro | `fal-ai/kling-video/v2.5-turbo/pro/image-to-video` |
| Kling 2.1 Pro | `fal-ai/kling-video/v2.1/standard/image-to-video` |
| Kling 1.6 | `fal-ai/kling-video/v1.6/pro/image-to-video` |
| Seedance 2.0 | `doubao-seedance-2-0-260128/i2v` |
| Seedance 2.0 Fast | `doubao-seedance-2-0-fast-260128/i2v` |
| Seedance 1.5 Pro | `fal-ai/bytedance/seedance/v1.5/pro/image-to-video` |
| Seedance 1.0 Pro | `doubao-seedance-1-0-pro` |
| Seedance 1.0 Lite | `fal-ai/bytedance/seedance/v1/lite/image-to-video` |
| Hailuo Video 2.3 Pro | `fal-ai/minimax/hailuo-2.3/pro/image-to-video` |
| Hailuo Video 02 | `fal-ai/minimax/hailuo-02/standard/image-to-video` |
| Wan 2.6 | `fal-ai/wan/v2.6/image-to-video` |
| Wan 2.5 | `fal-ai/wan-25-preview/image-to-video` |
| Wan 2.2 | `fal-ai/wan/v2.2-a14b/image-to-video` |
| Vidu Q3 | `fal-ai/vidu/q3/image-to-video` |
| Runway Gen-4 | `gen4_turbo` |
| Luma Ray 2 | `ray-2` |
| Grok Imagine Video | `xai/grok-imagine-video/image-to-video` |

硬规则：

- `videoMaker` 只能使用上表里明确映射到 `videoMaker` 的 id
- 不要把 text-to-video、video-to-video、motion-control 的 id 误填到 `videoMaker`
- 像 `Sora 2`、`Kling 3.0`、`Seedance 2.0` 这类推荐名必须先落到精确 id，再写入 `selectedModels`

---

## 模型选型

### Sora 2
- 电影级质量、动作自然
- 支持音画同步
- **UGC / 广告优先模型（首选）**
- 适合：真人出镜 / 高质量广告

---

### Veo 3.1 Fast
- 生成速度快、稳定
- 支持音画同步
- 适合：批量生成 / 快速测试

---

### Kling 3.0 / 2.6
- 中文能力强
- 支持音画同步
- 综合能力仅次于 Sora
- 适合：中文UGC / 批量广告

---

### Seedance 2.0
- 结构稳定、风格控制强
- 支持音画同步
- ❌ 不支持真人
- 适合：商品 / 场景 / 动画 / 非真人内容

---

### Hailuo Video 2.3
- 运镜能力极强（pan / zoom / cinematic）
- ❌ 不支持音画同步
- 适合：镜头表达 / 运镜增强

---

## 实际优先级（UGC / 广告）

```text
Sora 2 > Kling > Seedance 2.0 > Hailuo
```

---

## 关键约束

```text
真人 → 禁用 Seedance
需要音画同步 → 禁用 Hailuo
强调运镜 → 使用 Hailuo
```

---

## 模型能力对比

| 能力 | 支持模型 |
|---|---|
| 音画同步 | Sora 2, Veo 3.1, Kling, Seedance 2.0 |
| 中文能力 | Kling, Seedance |
| 运镜能力 | Hailuo |
| 高质量 | Sora |
| 高速度 | Veo |

---

## 参数（保留原结构）

### 通用参数

| 参数 | 说明 |
|---|---|
| aspect_ratio | 控制画幅 |
| duration | 视频时长 |

---

### Veo 3.1 Fast

| 参数 | 选项 |
|---|---|
| resolution | 720p / 1080p |
| duration | 4s / 6s / 8s |

---

### Sora 2

| 参数 | 选项 |
|---|---|
| aspect_ratio | 16:9 / 9:16 |
| duration | 8s / 12s / 16s / 20s |

---

### Kling

| 参数 | 选项 |
|---|---|
| aspect_ratio | 16:9 / 9:16 / 1:1 |
| audio_mode | 无音频 / 原生音频 / 语音控制 |
| duration | 3s–15s |

---

### Seedance 2.0

| 参数 | 选项 |
|---|---|
| aspect_ratio | 16:9 / 4:3 / 1:1 / 9:16 / 3:4 / 21:9 |
| resolution | 480p / 720p |
| duration | 4s–14s |
| generate_audio | on / off |

---

### Hailuo

| 参数 | 选项 |
|---|---|
| aspect_ratio | 16:9 / 9:16 / 1:1 |
| duration | 模型定义 |

---

## 时长选择（核心）

```text
单镜头时长 = 总时长 ÷ 镜头数
```

### 推荐

```text
4–6s → 最稳
6–8s → 常用
8–12s → 常用高质量区间（Sora / Seedance）
12s–20s → 仅建议使用更强模型（优先 Sora 2），风险明显上升
```

结论：

```text
优先多短镜头拼接，而不是长视频一次生成
```

---

## 比例选择（核心）

### 原则

```text
优先按投放场景选择
若图片构图强 → 优先继承原图
```

### 映射

```text
9:16 → 短视频（TikTok / Reels）
4:5 → 广告（部分平台）
16:9 → 横版视频
1:1 → 通用
```

---

## 核心结论

```text
Sora / Kling = UGC主力
Seedance = 非真人稳定内容
Hailuo = 运镜能力

时长：拆分短镜头
比例：由场景 + 图片共同决定
```
