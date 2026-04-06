# Node Catalog And Model Selection

Use this file when building or editing OpenCreator workflow graphs.

This file captures the hard platform constraints that should be checked before writing `nodes` and `edges`. For deeper generator-specific build patterns, start from the main `SKILL.md` and then read the referenced Step 1 to Step 4 files.

## Node Whitelist

Only create these node types unless the platform documentation is updated.

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

### Non-Executable Utility Nodes

- `assembleNow`
- `stickyNodesNode`
- `groupNode`

### Deprecated Or Legacy

- Do not create:
  - `syncVideoAudio`
  - `imageAnnotationNode`
  - `videoAnnotationNode`
- Legacy-only, not recommended for new workflows:
  - `describeImage`
  - `oneClickStyle`

## Pin Types And Compatibility

The platform has 6 pin types:

- `text`
- `image`
- `video`
- `audio`
- `subject`
- `style`

Compatibility rules:

- `text` -> `text`
- `image` -> `image`
- `image` -> `subject`
- `image` -> `style`
- `video` -> `video`
- `audio` -> `audio`

`subject` and `style` are aliases of image-like input positions, not standalone output types.

## Edge Validation Rules

Before adding an edge, confirm all of these:

1. `source !== target`
2. `sourceHandle` and `targetHandle` are type-compatible
3. The target pin has not exceeded its connection cap
4. The new edge does not create a cycle

## Handle Map

Use these input and output handles:

- `textInput`
  - out: `text`
- `imageInput`
  - out: `image`
- `videoInput`
  - out: `video`
- `audioInput`
  - out: `audio`
- `textGenerator`
  - in: `text,image,video,audio`
  - out: `text`
- `describeImage`
  - in: `image`
  - out: `text`
- `scriptSplit`
  - in: `text`
  - out: `text`
- `imageMaker`
  - in: `text`
  - out: `image`
- `imageToImage`
  - in: `image,text`
  - out: `image`
- `relight`
  - in: `image`
  - out: `image`
- `imageAngleControl`
  - in: `image`
  - out: `image`
- `imageUpscaler`
  - in: `image`
  - out: `image`
- `backgroundEditor`
  - in: `image,text`
  - out: `image`
- `textToVideo`
  - in: `text`
  - out: `video`
- `omniVideoGeneration`
  - in: `video,audio,image,text`
  - out: `video`
- `videoMaker`
  - in: `image,text`
  - out: `video`
- `videoToVideo`
  - in: `video,text,subject,style`
  - out: `video`
- `klingMotionControl`
  - in: `image,video,text`
  - out: `video`
- `videoLipSync`
  - in: `video,audio`
  - out: `video`
- `imageAudioToVideo`
  - in: `image,audio,text`
  - out: `video`
- `videoUpscaler`
  - in: `video`
  - out: `video`
- `textToSpeech`
  - in: `text`
  - out: `audio`
- `voiceCloner`
  - in: `audio,text`
  - out: `audio`
- `musicGenerator`
  - in: `text`
  - out: `audio`
- `assembleNow`
  - in: `video,audio,image`
  - out: `video`

## Connection Caps

Respect these hard limits:

- `scriptSplit.text = 1`
- `imageUpscaler.image = 1`
- `backgroundEditor.image = 1`
- `relight.image = 1`
- `imageAngleControl.image = 1`
- `videoUpscaler.video = 1`
- `videoToVideo.video = 1`
- `klingMotionControl.image = 1`
- `klingMotionControl.video = 1`
- `videoMaker.image <= 4`
- `omniVideoGeneration.image <= 9`
- `omniVideoGeneration.video <= 3`
- `omniVideoGeneration.audio <= 3`
- `videoLipSync.video = 1`
- `videoLipSync.audio = 1`
- `imageAudioToVideo.image = 1`
- `imageAudioToVideo.audio = 1`

If a pin is not listed here, no hard cap is currently documented.

## JSON Authoring Rules

- Input nodes are non-executable and should use `selectedModels: []`
- `textInput` stores content in `inputText`
- `imageInput` stores the media URL in `imageBase64`
- `audioInput` stores the media URL in `inputAudio`
- `videoInput` uses:
  - `inputVideo`
  - `inputVideoPoster`
  - `inputVideoDuration`
- For executable nodes, keep `selectedModels` non-empty when the node requires model selection
- Use documented defaults for `model_options` or `modelConfigs` when they exist
- For list-style patterns, preserve the actual graph semantics through wiring rather than inventing special list node types

## Model ID Safety Rules

- The `Confirmed model IDs` section inside each Step 3 file is the source of truth for exact model IDs
- Only place a model ID in `selectedModels` if that exact ID is mapped to the current node type / atom
- Never infer or synthesize a model ID from a display name such as `Sora 2`, `Kling 3.0`, `GPT Image 1.5`, or `Seedream 5.0 Lite`
- If a node is fixed-mode or fixed-provider in the docs below, follow that documented behavior instead of inventing alternate `selectedModels`
- If a node has no dedicated Step 3 file, use the node-specific rules in this file as the fallback source of truth
- If a node still has no confirmed model-selection entry after checking those docs, prefer the documented default in this file; if there is no confirmed default, stop and ask rather than guessing

## Important Node Rules

### Input Nodes

#### `textInput`

- Category:
  - Input
- Output:
  - `text`
- Key field:
  - `inputText`
- Valid when:
  - The HTML-stripped text is still non-empty
- Model rule:
  - `selectedModels` must be `[]`

#### `imageInput`

- Category:
  - Input
- Output:
  - `image`
- Key field:
  - `imageBase64`
- Valid when:
  - URL is non-empty
- Model rule:
  - `selectedModels` must be `[]`

#### `audioInput`

- Category:
  - Input
- Output:
  - `audio`
- Key field:
  - `inputAudio`
- Model rule:
  - `selectedModels` must be `[]`

#### `videoInput`

- Category:
  - Input
- Output:
  - `video`
- Key fields:
  - `inputVideo`
  - `inputVideoPoster`
  - `inputVideoDuration`
- Model rule:
  - `selectedModels` must be `[]`

### Text Nodes

#### `textGenerator`

- Input:
  - `text` required
  - `image` optional
  - `video` optional
  - `audio` optional
- Output:
  - `text`
- Default model:
  - `openai/gpt-4o-mini`
- Default `model_options`:

```json
{
  "attachments": []
}
```

- Validation:
  - `selectedModels.length > 0`
  - Either `inputText` has content or valid upstream inputs exist

#### `scriptSplit`

- Input:
  - `text` required
- Output:
  - `text`
- Connection cap:
  - `text <= 1`
- Default model:
  - `openai/gpt-5.2`
- Validation:
  - Provide `inputText` or exactly one upstream text input

### Image Nodes

#### `imageMaker`

- Input:
  - `text` required
- Output:
  - `image`
- Default model:
  - `minimax/hailuo-image-01`
- Lens defaults:
  - `lensStyleEnabled = false`
  - `lensStyle.camera_style = "none"`
  - `lensStyle.lens_preset = "none"`
  - `lensStyle.focal_length = "none"`
  - `lensStyle.lighting_style = "none"`
- Validation:
  - `selectedModels.length > 0`
  - `inputText` exists or an upstream text source exists

#### `imageToImage`

- Input:
  - `image` required
  - `text` required
- Output:
  - `image`
- Default model:
  - `fal-ai/gemini-flash-edit/multi`
- Rule:
  - If `inputText` already has content, the text pin requirement is considered satisfied
- Validation:
  - `selectedModels.length > 0`

#### `backgroundEditor`

- Input:
  - `image` required
  - `text` optional
- Output:
  - `image`
- Default `model_options`:

```json
{
  "model_mode": "Change BG"
}
```

- Confirmed fixed modes:
  - `Change BG` -> `Fal ICLight V2`
  - `Remove BG` -> `Fal Bria Background Remove`
- Rule:
  - If `model_mode` is `Change BG`, `inputText` must be non-empty
  - If `model_mode` is `Remove BG`, `inputText` may be omitted
  - Do not invent alternate model IDs for this node

#### `relight`

- Input:
  - `image` required
- Output:
  - `image`
- Default model:
  - `gemini-3-pro-image-preview`
- Confirmed model IDs:
  - `gemini-3-pro-image-preview`

#### `imageAngleControl`

- Input:
  - `image` required
- Output:
  - `image`
- Default model:
  - `fal-ai/qwen-image-edit-2511-multiple-angles`
- Confirmed model IDs:
  - `fal-ai/qwen-image-edit-2511-multiple-angles`
- Validation:
  - `selectedModels.length > 0`

### Video Nodes

#### `videoMaker`

- Input:
  - `image` required
  - `text` optional
- Output:
  - `video`
- Image cap:
  - 4
- Default model:
  - `fal-ai/bytedance/seedance/v1/lite/image-to-video`
- Special rule:
  - If result images are connected, the node may switch to `multi_ingredients` style behavior
- Validation:
  - `selectedModels.length > 0`
  - At least one valid image input exists

#### `textToVideo`

- Input:
  - `text` required
- Output:
  - `video`
- Default model:
  - `fal-ai/minimax/hailuo-02/standard/text-to-video`

#### `omniVideoGeneration`

- Input:
  - `text` required
  - `image` optional up to 9
  - `video` optional up to 3
  - `audio` optional up to 3
- Output:
  - `video`
- Default model:
  - `doubao-seedance-2-0-260128/i2v`
- Confirmed model IDs:
  - `doubao-seedance-2-0-260128/i2v`
  - `doubao-seedance-2-0-fast-260128/i2v`
- Validation:
  - `selectedModels.length > 0`
  - Text must be satisfied by `inputText` or an upstream text input

#### `videoToVideo`

- Input:
  - `video` required
  - `text` required
  - `subject` optional
  - `style` optional
- Output:
  - `video`
- Default model:
  - `fal-ai/kling-video/o3/standard/video-to-video`
- Confirmed model IDs:
  - `fal-ai/kling-video/o1/video-to-video`
  - `fal-ai/kling-video/o3/standard/video-to-video`

#### `videoLipSync`

- Input:
  - `video` required
  - `audio` required
- Output:
  - `video`
- Default model:
  - `fal-ai/pixverse/lipsync`
- Confirmed model IDs:
  - `fal-ai/pixverse/lipsync`
  - `fal-ai/sync-lipsync/v2`

#### `klingMotionControl`

- Input:
  - `image` required
  - `video` required
  - `text` optional
- Output:
  - `video`
- Default model:
  - `fal-ai/kling-video/v2.6/standard/motion-control`
- Confirmed model IDs:
  - `fal-ai/kling-video/v2.6/standard/motion-control`
  - `fal-ai/kling-video/v2.6/pro/motion-control`
- Special rule:
  - If `character_orientation` is `image`, upstream video duration must be <= `10000` ms
  - Otherwise upstream video duration must be <= `30000` ms
- Validation:
  - `selectedModels.length > 0`

#### `imageAudioToVideo`

- Input:
  - `image` required
  - `audio` required
  - `text` required
- Output:
  - `video`
- Default model:
  - `fal-ai/infinitalk`
- Confirmed model IDs:
  - `fal-ai/infinitalk`
- Rule:
  - If `inputText` already has content, the text pin requirement is considered satisfied

### Audio Nodes

#### `textToSpeech`

- Input:
  - `text` required
- Output:
  - `audio`
- Default model:
  - `fish-audio/speech-1.6`
- Confirmed model IDs:
  - `fish-audio/speech-1.6`
  - `fal-ai/elevenlabs/tts/multilingual-v2`
  - `fal-ai/minimax/speech-2.8-hd`
  - `fal-ai/minimax/speech-2.8-turbo`
- Additional requirement:
  - A voice must also be selected through model config or selected voices

#### `musicGenerator`

- Input:
  - `text` required
- Output:
  - `audio`
- Default model:
  - Empty array
- Default `model_options`:

```json
{
  "make_instrumental": false
}
```

- Model rule:
  - No stable `selectedModels` entry is currently documented for this node; do not invent a model ID for this node

#### `voiceCloner`

- Input:
  - `audio` required
  - `text` required
- Output:
  - `audio`
- Default model:
  - `fal-ai/qwen-3-tts/clone-voice/1.7b`
- Confirmed model IDs:
  - `fal-ai/qwen-3-tts/clone-voice/1.7b`
  - `fal-ai/minimax/voice-clone`
- Validation:
  - `selectedModels.length > 0`

## Model Selection Strategy

Use this file for hard defaults and safety checks. For nuanced model choice by task quality, read the dedicated Step 3 model-selection files referenced from the main `SKILL.md`.

At minimum:

- Respect node-level default models when no stronger instruction is available
- Treat these nodes as requiring strict `selectedModels` validation:
  - `textGenerator`
  - `describeImage`
  - `imageMaker`
  - `imageToImage`
  - `imageAngleControl`
  - `videoMaker`
  - `omniVideoGeneration`
  - `klingMotionControl`
  - `voiceCloner`

## Node Data Minimum Structure

### Executable Node

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

### Input Node

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

Do not write derived or front-end-only fields such as `isNodeConnected`, `isTextPinConnected`, `isImagePinConnected`, `estimatedTime`, or `local_file`.

## Node / Edge JSON Templates

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

ID format: `{nodeType}-{timestamp}-{nanoid(6)}`

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

ID format: `customEdge-{sourceId}-{targetId}-{sourceHandle}-{targetHandle}`

## Layout Hints (DAG)

- `x = 100 + layer * 400`
- `y = 100 + indexInLayer * 300`
- `layer` starts at 0 for input nodes and increments rightward

## Review Checklist

Before trusting a graph, confirm:

- Every node type is on the whitelist
- Every edge obeys pin compatibility
- Every capped pin stays within limits
- Non-executable input nodes use `selectedModels: []`
- Executable nodes that require model selection have non-empty `selectedModels`
- Required media and text fields are actually present
