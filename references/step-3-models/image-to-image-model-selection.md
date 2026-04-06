# Image to Image – Model Selection Knowledge

## Overview

Image to Image 用于基于已有图片进行视觉变换。

与 Text-to-Image 不同，它以**参考图像**为起点，在保留结构、构图或主体的前提下，对风格、细节或内容进行修改。

---

## Input modalities

| Modality | Description | Limits |
|---|---|---|
| **Image (required)** | 输入参考图 | JPEG/PNG，≤10MB |
| **Text (required)** | 变换指令（如何改） | — |

> 注意：结果由“原图 + 指令”共同决定，如果冲突较大，结果会不稳定。

---

## Output modality

| Modality | Description |
|---|---|
| **Image** | 转换后的图像 |

---

## Confirmed model IDs

只使用下面这些已确认归属于 `imageToImage` 的模型 ID。

| 展示名 | 精确 model id |
|---|---|
| Banana Pro | `gemini-3-pro-image-preview` |
| Banana Pro VIP | `gemini-3-pro-image-preview-vip` |
| Nano Banana | `fal-ai/nano-banana/edit` |
| Nano Banana 2 | `fal-ai/gemini-3.1-flash-image-preview/edit` |
| Nano Banana 2 VIP | `fal-ai/gemini-3.1-flash-image-preview-vip/edit` |
| GPT Image 1.5 | `fal-ai/gpt-image-1.5/edit` |
| GPT Image 1 | `openai/gpt-image-1` |
| Flux Kontext | `fal-ai/flux-pro/kontext/multi` |
| Flux 2 Pro | `fal-ai/flux-2-pro/edit` |
| Gemini 2.0 Flash | `fal-ai/gemini-flash-edit/multi` |
| Qwen Image Edit | `fal-ai/qwen-image-edit-plus` |
| Seedream 5.0 | `fal-ai/bytedance/seedream/v5/edit` |
| Seedream 5.0 Lite | `fal-ai/bytedance/seedream/v5/lite/edit` |
| Seedream 4.5 | `fal-ai/bytedance/seedream/v4.5/edit` |
| Seedream 4.0 | `fal-ai/bytedance/seedream/v4/edit` |
| Grok Imagine | `xai/grok-imagine-image/edit` |

硬规则：

- `imageToImage` 只能从上表选择 model id
- 不要把 text-to-image 的 id 误填到 `imageToImage`
- 不要把展示名、品牌名或版本名当作 `selectedModels` 的值

---

## Model selection and parameters

---

### Banana Pro

**特点：**  
强写实编辑能力，高保真细节保留

#### Parameters

| Parameter | Options | Description |
|---|---|---|
| `aspect_ratio` | `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9` | 控制输出画幅 |
| `resolution` | `1K`, `2K`, `4K` | 控制输出分辨率 |

---

### Seedream 5.0 Lite

**特点：**  
强风格化与设计表达能力

#### Parameters

| Parameter | Options | Description |
|---|---|---|
| `image_size` | `square_hd`, `square`, `portrait_4_3`, `portrait_16_9`, `landscape_4_3`, `landscape_16_9`, `auto_2K`, `auto_3K` | 控制比例与分辨率 |

---

### GPT Image 1.5

**特点：**  
编辑可控性强，结构稳定

#### Parameters

| Parameter | Options | Description |
|---|---|---|
| `size` | `1024×1024`, `1536×1024`, `1024×1536`, `auto` | 输出尺寸 |
| `quality` | `low`, `medium`, `high` | 渲染质量 |
| `background` | `transparent`, `opaque`, `auto` | 背景类型 |

---

## Model Selection（核心）

### 按修改目标选择

```text
高保真 / 写实编辑 → Banana Pro
风格迁移 / 设计表达 → Seedream
结构控制 / 精准修改 → GPT Image
```

---

### 按“改动强度”选择

```text
轻修改（微调） → GPT Image
中等修改（风格 + 局部） → Seedream
重修改（重渲染 / 高真实感） → Banana
```

---

## Aspect Ratio Selection（补充）

### 核心原则

```text
优先继承原图比例
```

因为：
- 原图已经包含构图信息
- 强改比例会破坏结构稳定性

---

### 何时修改比例

仅在以下情况：

```text
需要适配投放场景（如 9:16 / 4:5）
需要重构画面（重新构图）
```

---

### 参数映射

不同模型控制方式：

```text
Banana → aspect_ratio
Seedream → image_size
GPT Image → size
```

本质一致：控制画幅

---

## Key transformation patterns

- 风格迁移（style transfer）  
- 内容修改（content edit）  
- 细节增强（enhancement）  
- 变体生成（variation）  
- 重构（recomposition）  

---

## Common use cases

- 商品图优化  
- UGC质量提升  
- 创意变体生成  
- 人物一致性维护  
- 广告素材迭代  

---

## Limitations

- 强依赖输入图质量  
- 不同模型可控性差异明显  
- 复杂修改可能不稳定  
- 指令与图像冲突时结果不可控  

---

## 核心结论

```text
Image-to-Image = 在“原图约束”下进行修改

Banana → 写实重建
Seedream → 风格表达
GPT Image → 精准控制

比例默认继承原图，除非有明确重构需求
```
