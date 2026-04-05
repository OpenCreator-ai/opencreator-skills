# OpenCreator Create-Workflow 参考（2026-04）

本文件用于 `create-workflow` skill 生成标准 `nodes/edges`。

---

## 1) 节点白名单

### Input

- `textInput`
- `imageInput`
- `videoInput`
- `audioInput`

### Text

- `textGenerator`
- `scriptSplit`

### Image

- `imageMaker`
- `imageToImage`
- `relight`
- `imageAngleControl`
- `imageUpscaler`
- `backgroundEditor`

### Video

- `textToVideo`
- `omniVideoGeneration`
- `videoMaker`
- `videoToVideo`
- `klingMotionControl`
- `videoLipSync`
- `imageAudioToVideo`
- `videoUpscaler`

### Audio

- `textToSpeech`
- `musicGenerator`
- `voiceCloner`

### Handy（不可执行，不放执行主链路）

- `assembleNow`
- `stickyNodesNode`
- `groupNode`

### 废弃节点（不应再创建）

- `syncVideoAudio`
- `imageAnnotationNode`
- `videoAnnotationNode`

### 兼容旧流（新建不推荐）

- `describeImage`
- `oneClickStyle`

---

## 2) Pin 类型与连线规则

系统共 6 种 Pin 类型：`text`、`image`、`video`、`audio`、`subject`、`style`

类型兼容规则：

| 源输出  | 可连接到的目标输入          |
| ------- | --------------------------- |
| `text`  | `text`                      |
| `image` | `image`、`subject`、`style` |
| `video` | `video`                     |
| `audio` | `audio`                     |

`subject` 和 `style` 是 `image` 的别名，不是独立输出类型。

### 建边前 4 项校验

1. 不能自连：`source !== target`
2. `sourceHandle` 与 `targetHandle` 必须类型兼容
3. 目标 Pin 未超过最大连线数
4. 加这条边之后不能形成环

---

## 3) 句柄映射（Pin）

- `textInput`: out `text`
- `imageInput`: out `image`
- `videoInput`: out `video`
- `audioInput`: out `audio`
- `textGenerator`: in `text,image,video,audio`; out `text`
- `describeImage`: in `image`; out `text`
- `scriptSplit`: in `text`; out `text`
- `imageMaker`: in `text`; out `image`
- `imageToImage`: in `image,text`; out `image`
- `relight`: in `image`; out `image`
- `imageAngleControl`: in `image`; out `image`
- `imageUpscaler`: in `image`; out `image`
- `backgroundEditor`: in `image,text`; out `image`
- `textToVideo`: in `text`; out `video`
- `omniVideoGeneration`: in `video,audio,image,text`; out `video`
- `videoMaker`: in `image,text`; out `video`
- `videoToVideo`: in `video,text,subject,style`; out `video`
- `klingMotionControl`: in `image,video,text`; out `video`
- `videoLipSync`: in `video,audio`; out `video`
- `imageAudioToVideo`: in `image,audio,text`; out `video`
- `videoUpscaler`: in `video`; out `video`
- `textToSpeech`: in `text`; out `audio`
- `voiceCloner`: in `audio,text`; out `audio`
- `musicGenerator`: in `text`; out `audio`
- `assembleNow`: in `video,audio,image`; out `video`（不可执行）

---

## 4) 连接上限

- `scriptSplit.text = 1`
- `imageUpscaler.image = 1`
- `backgroundEditor.image = 1`
- `relight.image = 1`
- `imageAngleControl.image = 1`
- `videoUpscaler.video = 1`
- `videoToVideo.video = 1`
- `klingMotionControl.image = 1, video = 1`
- `videoMaker.image <= 4`
- `omniVideoGeneration.image <= 9, video <= 3, audio <= 3`
- `videoLipSync.video = 1, audio = 1`
- `imageAudioToVideo.image = 1, audio = 1`

未列出的 Pin 无硬性上限。

---

## 5) 节点详细规格

### 5.1 输入节点

输入节点不执行，只提供上游原始数据。`selectedModels` 必须为 `[]`。

**`textInput`**

- 输出：`text`
- 关键字段：`inputText`（富文本 HTML 字符串）
- 有效条件：`inputText` 去除 HTML 标签后仍有非空文本

**`imageInput`**

- 输出：`image`
- 关键字段：`imageBase64`（图片 URL）
- 有效条件：`imageBase64` 非空

**`audioInput`**

- 输出：`audio`
- 关键字段：`inputAudio`（音频 URL）
- 有效条件：`inputAudio` 非空

**`videoInput`**

- 输出：`video`
- 关键字段：`inputVideo`（视频 URL）、`inputVideoPoster`（封面）、`inputVideoDuration`（时长，毫秒）
- 有效条件：`inputVideo` 非空

### 5.2 文本类任务节点

**`textGenerator`**

- 输入：`text` 必填，`image` 可选，`video` 可选，`audio` 可选
- 输出：`text`
- 最大输入连线：无限制
- 默认模型：`["openai/gpt-4o-mini"]`
- 默认 `model_options`：`{ "attachments": [] }`
- 运行要求：`selectedModels.length > 0`；`inputText` 有内容或有满足要求的上游输入
- **需要强制 selectedModels 校验**

**`scriptSplit`**

- 输入：`text` 必填
- 输出：`text`
- 最大输入连线：`text` 最多 1 条
- 默认模型：`["openai/gpt-5.2"]`
- 默认 `model_options`：`{ "attachments": [] }`
- 运行要求：应提供 `inputText` 或 1 条 `text` 上游

**`describeImage`**（兼容旧流，新建不推荐）

- 输入：`image` 必填
- 输出：`text`
- 最大输入连线：无限制
- **需要强制 selectedModels 校验**

### 5.3 图像类任务节点

**`imageMaker`**

- 输入：`text` 必填
- 输出：`image`
- 最大输入连线：无限制
- 默认模型：`["minimax/hailuo-image-01"]`
- 附加字段：`lensStyleEnabled`=`false`，`lensStyle.camera_style`=`"none"`，`lensStyle.lens_preset`=`"none"`，`lensStyle.focal_length`=`"none"`，`lensStyle.lighting_style`=`"none"`
- 运行要求：`selectedModels.length > 0`；`inputText` 有内容或有 `text` 上游
- **需要强制 selectedModels 校验**

**`imageToImage`**

- 输入：`image` 必填，`text` 必填
- 输出：`image`
- 最大输入连线：无限制
- 默认模型：`["fal-ai/gemini-flash-edit/multi"]`
- 附加字段：`lensStyleEnabled`=`false`，`lensStyle.*`=`"none"` 系列
- 特殊规则：如果 `inputText` 本身有内容，`text` 必填 Pin 视为已满足
- **需要强制 selectedModels 校验**

**`imageUpscaler`**

- 输入：`image` 必填
- 输出：`image`
- 最大输入连线：`image` 最多 1 条
- 默认 `model_options`：`{ "upscale_factor": "2" }`
- 允许值：`"2"` / `"4"` / `"6"`

**`backgroundEditor`**

- 输入：`image` 必填，`text` 可选
- 输出：`image`
- 最大输入连线：`image` 最多 1 条
- 默认 `model_options`：`{ "model_mode": "Change BG" }`
- 特殊规则：`model_mode === "Change BG"` 时 `inputText` 不能为空；`model_mode === "Remove BG"` 时可无 `inputText`

**`relight`**

- 输入：`image` 必填
- 输出：`image`
- 最大输入连线：`image` 最多 1 条
- 默认模型：`["gemini-3-pro-image-preview"]`
- 默认 `modelConfigs`：

```json
{
  "gemini-3-pro-image-preview": {
    "light_x": 45,
    "light_y": 30,
    "light_color": "#fffbe6",
    "light_brightness": 70,
    "light_temperature": 5000,
    "light_quality": "product_studio",
    "aspect_ratio": "16:9"
  }
}
```

**`imageAngleControl`**

- 输入：`image` 必填
- 输出：`image`
- 最大输入连线：`image` 最多 1 条
- 默认模型：`["fal-ai/qwen-image-edit-2511-multiple-angles"]`
- 默认 `modelConfigs`：

```json
{
  "fal-ai/qwen-image-edit-2511-multiple-angles": {
    "horizontal_angle": 0,
    "vertical_angle": 0,
    "zoom": 5,
    "lora_scale": 1
  }
}
```

- **需要强制 selectedModels 校验**

### 5.4 视频类任务节点

**`videoMaker`**

- 输入：`image` 必填，`text` 可选
- 输出：`video`
- 最大输入连线：`image` 最多 4 条
- 默认模型：`["fal-ai/bytedance/seedance/v1/lite/image-to-video"]`
- 特殊规则：总图片连接数 = `imageInput` 数量 + 结果型图片原子数量；有结果型图片原子连接时使用 `multi_ingredients`
- 特殊规则：当已选模型不支持当前图片数量时，前端会自动清理不兼容模型；如果全部被移除，会回退到默认模型 `fal-ai/bytedance/seedance/v1/lite/image-to-video`
- 运行要求：`selectedModels.length > 0`；至少要有图片输入
- **需要强制 selectedModels 校验**

**`textToVideo`**

- 输入：`text` 必填
- 输出：`video`
- 最大输入连线：无限制
- 默认模型：`["fal-ai/minimax/hailuo-02/standard/text-to-video"]`

**`omniVideoGeneration`**

- 输入：`text` 必填，`image` 可选，`video` 可选，`audio` 可选
- 输出：`video`
- 最大输入连线：`image` 最多 9 条，`video` 最多 3 条，`audio` 最多 3 条
- 默认模型：`["doubao-seedance-2-0-260128/i2v"]`
- 默认 `modelConfigs`：

```json
{
  "doubao-seedance-2-0-260128/i2v": {
    "aspect_ratio": "16:9",
    "generate_audio": false,
    "resolution": "720p",
    "duration": "8"
  }
}
```

- 运行要求：`selectedModels.length > 0`；`text` 必须由本节点 `inputText` 或上游 `text` 输入满足
- **需要强制 selectedModels 校验**

**`videoToVideo`**

- 输入：`video` 必填，`text` 必填，`subject` 可选，`style` 可选
- 输出：`video`
- 最大输入连线：`video` 最多 1 条
- 默认模型：`["fal-ai/kling-video/o3/standard/video-to-video"]`
- 默认 `model_options`：`{ "node_mode": "edit-short", "duration": 5, "keep_audio": true }`

**`videoLipSync`**

- 输入：`video` 必填，`audio` 必填
- 输出：`video`
- 最大输入连线：`video` 1 条，`audio` 1 条
- 默认模型：`["fal-ai/pixverse/lipsync"]`
- 默认 `model_options`：`{ "loop_mode": "Cut-off" }`

**`videoUpscaler`**

- 输入：`video` 必填
- 输出：`video`
- 最大输入连线：`video` 最多 1 条
- 默认模型：`["fal-ai/topaz/upscale/video"]`
- 默认 `model_options`：`{ "upscale_factor": "2", "frames_per_second": 24 }`

**`klingMotionControl`**

- 输入：`image` 必填，`video` 必填，`text` 可选
- 输出：`video`
- 最大输入连线：`image` 1 条，`video` 1 条
- 默认模型：`["fal-ai/kling-video/v2.6/standard/motion-control"]`
- 默认 `modelConfigs`：

```json
{
  "fal-ai/kling-video/v2.6/standard/motion-control": {
    "character_orientation": "video",
    "keep_original_sound": true
  }
}
```

- 特殊规则：`character_orientation === "image"` 时上游视频时长不超过 `10000` ms，否则不超过 `30000` ms
- **需要强制 selectedModels 校验**

**`imageAudioToVideo`**

- 输入：`image` 必填，`audio` 必填，`text` 必填
- 输出：`video`
- 最大输入连线：`image` 1 条，`audio` 1 条
- 默认模型：`["fal-ai/infinitalk"]`
- 特殊规则：如果 `inputText` 本身有内容，`text` 必填 Pin 视为已满足

### 5.5 音频类任务节点

**`textToSpeech`**

- 输入：`text` 必填
- 输出：`audio`
- 最大输入连线：无限制
- 默认模型：`["fish-audio/speech-1.6"]`
- 默认 `modelConfigs`：

```json
{
  "fish-audio/speech-1.6": {
    "voice_ids": ["Elon_Musk"]
  }
}
```

- 特殊运行规则：除了模型，还必须有已选语音。满足下列任一：`model_options[modelId].selected_voices` 非空，或 `modelConfigs[modelId].voice_ids` 非空

**`musicGenerator`**

- 输入：`text` 必填
- 输出：`audio`
- 最大输入连线：无限制
- 默认模型：`[]`（空数组）
- 默认 `model_options`：`{ "make_instrumental": false }`

**`voiceCloner`**

- 输入：`audio` 必填，`text` 必填
- 输出：`audio`
- 最大输入连线：无限制
- 默认模型：`["fal-ai/qwen-3-tts/clone-voice/1.7b"]`
- **需要强制 selectedModels 校验**

### 5.6 不可执行节点

**`assembleNow`**

- 输入：`video` 可选，`audio` 可选，`image` 可选
- 输出：`video`（有输出 Pin 但不参与自动执行）
- 关键字段：`assembleAssets`=`[]`，`assemblePayload`=`""`

**`stickyNodesNode`**

- 输入：无 / 输出：无
- 作用：便签/视觉标注
- 关键字段：`inputText`=`""`，`backgroundColor`=`"#DDEEDB"`，`stickyMode`=`"text"`

**`groupNode`**

- 输入：无 / 输出：无
- 作用：分组/视觉容器
- 关键字段：`label`，`inputHint`，`backgroundColor`

---

## 6) 需要强制 selectedModels 校验的节点

前端运行校验会硬性检查 `selectedModels.length > 0` 的节点：

- `textGenerator`
- `imageMaker`
- `describeImage`
- `videoMaker`
- `omniVideoGeneration`
- `imageToImage`
- `klingMotionControl`
- `imageAngleControl`
- `voiceCloner`

其他任务节点虽然不会被前端硬性检查，但从保守建图角度仍建议提供合理的 `selectedModels`。

---

## 7) 特殊运行规则汇总

| 节点                 | 规则                                                                                                                         |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `textToSpeech`       | 必须有已选语音（`voice_ids` 或 `selected_voices`）                                                                           |
| `backgroundEditor`   | `model_mode === "Change BG"` 时 `inputText` 不能为空                                                                         |
| `imageToImage`       | 如果自己 `inputText` 有效，可跳过 text 必填 Pin                                                                              |
| `imageAudioToVideo`  | 如果自己 `inputText` 有效，可跳过 text 必填 Pin                                                                              |
| `videoMaker`         | 总图片连接数 = `imageInput` + 结果型图片原子；有结果型图片原子连接时使用 `multi_ingredients`，并按图片数量自动过滤不兼容模型 |
| `klingMotionControl` | `character_orientation === "image"` 时视频时长 <= 10s，否则 <= 30s                                                           |

---

## 8) data 最小结构

### 任务节点

```json
{
  "label": "Node Name",
  "description": "Node Description",
  "themeColor": "#73ADFF",
  "modelCardColor": "#73ADFF",
  "selectedModels": ["model-id"],
  "inputText": "",
  "imageBase64": "",
  "inputAudio": "",
  "inputVideo": "",
  "status": "idle",
  "isSelectMode": false,
  "workflowId": "<当前workflowId>"
}
```

### 输入节点

```json
{
  "label": "Node Name",
  "description": "Node Description",
  "themeColor": "#73ADFF",
  "selectedModels": [],
  "inputText": "",
  "imageBase64": "",
  "inputAudio": "",
  "inputVideo": "",
  "status": "idle",
  "workflowId": "<当前workflowId>"
}
```

不要写入 `isNodeConnected`、`isTextPinConnected`、`isImagePinConnected`、`estimatedTime`、`local_file` 等前端派生/应忽略字段。

---

## 9) Node/Edge 模板

### Node

```json
{
  "id": "textGenerator-1710000000000-a1b2c3",
  "type": "textGenerator",
  "position": { "x": 500, "y": 100 },
  "selected": false,
  "data": {
    "label": "Text Generator",
    "description": "Generate high-quality text",
    "themeColor": "#9DFF9E",
    "modelCardColor": "#9DFF9E",
    "selectedModels": ["openai/gpt-4o-mini"],
    "inputText": "",
    "imageBase64": "",
    "inputAudio": "",
    "inputVideo": "",
    "status": "idle",
    "isSelectMode": false,
    "workflowId": "<当前workflowId>",
    "model_options": { "attachments": [] }
  }
}
```

ID 格式：`{nodeType}-{timestamp}-{nanoid(6)}`

### Edge

```json
{
  "id": "customEdge-textInput-1-imageMaker-1-text-text",
  "source": "textInput-1",
  "target": "imageMaker-1",
  "sourceHandle": "text",
  "targetHandle": "text",
  "type": "customEdge"
}
```

ID 格式：`customEdge-{sourceId}-{targetId}-{sourceHandle}-{targetHandle}`

---

## 10) 布局建议（DAG）

- `x = 100 + layer * 400`
- `y = 100 + indexInLayer * 300`
- `layer` 从输入节点开始向右递增
