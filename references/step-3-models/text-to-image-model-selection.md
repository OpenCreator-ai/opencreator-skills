# Text-to-Image – Model Selection Knowledge

## Overview

Text-to-Image 用于将自然语言描述转化为图像。

它基于对主体、环境和风格的描述生成视觉内容。  
不同模型在能力和参数控制上存在差异。

---

## Input modality

| Modality | Description | Limitations |
|---|---|---|
| **Text (required)** | 图像描述（主体 / 场景 / 风格等） | 建议简洁明确 |

---

## Output modality

| Modality | Description |
|---|---|
| **Image** | 生成图像（PNG / JPEG / WEBP） |

---

## Available models and parameters

### Banana Pro 

**Strengths:**  
- 写实能力强（Photorealistic）  
- 高分辨率输出  
- 光影与物理结构表现优秀  

#### Parameters

| Parameter | Options | Description |
|---|---|---|
| `aspect_ratio` | `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9` | 控制画面比例 |
| `resolution` | `1K`, `2K`, `4K` | 控制输出分辨率 |

#### Notes

- 分辨率定义短边长度  
- 分辨率越高 → 质量更好 + 时间更长  

---

### Seedream 5.0 Lite

**Strengths:**  
- 风格化能力强  
- 构图与结构理解优秀  
- 更适合设计感视觉  

#### Parameters

| Parameter | Options | Description |
|---|---|---|
| `image_size` | `square_hd`, `square`, `portrait_4_3`, `portrait_16_9`, `landscape_4_3`, `landscape_16_9`, `auto_2K`, `auto_3K` | 预设比例与分辨率 |
| `enable_safety_checker` | true / false | 安全过滤 |

#### Notes

- 生成时间约 20–40s / 张  

---

### GPT Image 1.5

**Strengths:**  
- Prompt 遵循度高  
- 输出干净稳定  
- 支持透明背景  

#### Parameters

| Parameter | Options | Description |
|---|---|---|
| `size` | `1024×1024`, `1536×1024`, `1024×1536`, `auto` | 输出尺寸 |
| `quality` | `low`, `medium`, `high`, `auto` | 质量与成本控制 |
| `format` | `png`, `jpeg`, `webp` | 文件格式 |
| `background` | `transparent`, `opaque`, `auto` | 背景类型 |

#### Notes

- 最大分辨率低于 Banana Pro  
- 支持透明背景  

---

## Model Selection（核心）

### 按视觉目标选择

```text
写实 → Banana Pro
风格 / 设计 → Seedream
结构稳定 / UI → GPT Image
```

---

### 按复杂度选择

```text
复杂构图 → Seedream
高真实感 → Banana
简单稳定 → GPT Image
```

---

## Aspect Ratio Selection（新增）

### 核心原则

```text
比例由使用场景倒推
```

---

### 常见映射

```text
1:1 → 商品主图
4:5 → feed / 广告（最通用）
9:16 → 短视频封面 / TikTok
16:9 → 横版内容 / YouTube
```

---

### 与模型参数的关系

不同模型的比例控制方式：

- Banana → `aspect_ratio`
- Seedream → `image_size`
- GPT Image → `size`

👉 本质都是在控制同一件事：**画幅结构**

---

## Common use cases

- 商品图  
- 社交媒体视觉  
- 广告素材  
- 概念设计  
- UI / 图形  

---

## Limitations

- 模型能力差异明显  
- 高分辨率增加成本与延迟  
- 参数体系不统一（不同模型字段不同）  
- 安全策略可能限制输出  

---

## 核心结论

```text
模型决定“生成能力”
参数决定“输出控制”
比例决定“使用场景与构图”
```
