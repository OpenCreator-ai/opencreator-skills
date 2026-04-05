# Text Prompt Generation Best Practices (Knowledge Base)

This document defines best practices for using a text generation model to produce downstream prompts, structured descriptions, scripts, and analysis outputs inside a creative workflow.

This knowledge base is designed for scenarios where the text model acts as the planning and specification layer for other generation blocks.

It is especially relevant for:
- image prompt generation
- video prompt generation
- storyboard generation
- multi-image set generation
- script reconstruction from video analysis
- structured scene-by-scene content planning

> Important:
>
> When the goal is to generate prompts for image or video models, the text model should follow the corresponding dedicated guides:
>
> - Image Prompt Best Practices
> - Video Prompt Best Practices
>
> This document focuses on how to generate good text outputs for downstream use, not on replacing the model-specific image/video prompt guides.

---

## 1. Core Output Rules

These rules should be treated as mandatory defaults unless the user explicitly asks otherwise.

### 1.1 Output results only

The generated text must:
- not explain what it is doing
- not include reasoning
- not include transition phrases
- not include conversational filler
- not include summaries before the result
- not include phrases like:
  - “Here is the result”
  - “Sure, I can help with that”
  - “Below is the output”
  - “Let’s break it down”
  - “This scene shows”
  - “The following is”

The model should output the result directly.

### 1.2 Keep each numbered block concise

Each numbered block must be independently usable.

Rules:
- each numbered description should stay compact
- each numbered block should be no longer than **1500 Chinese characters or equivalent length**
- if the content is long, split it into more numbered blocks instead of making one block too long
- one numbered block should describe one execution target only

### 1.3 One block = one target

A strong rule:
- one image description = one image target
- one shot description = one shot target
- one scene description = one scene target

Do not merge multiple targets into one block.

### 1.4 Make outputs splitter-friendly

If the output may be passed into Text Splitter, the model should generate content in a stable format.

Best practices:
- use explicit numbering
- keep each item self-contained
- avoid mixing multiple scenes inside one numbered item
- keep labels stable

Good formats:
- `Image 01`
- `Shot 01`
- `Scene 01`
- JSON with stable keys

---

## 2. Role Assignment by Generation Scenario

Before generating content, the text model should identify the user’s scenario and assign itself a concrete professional role.

The role must:
- match the task type
- establish the right domain perspective
- influence vocabulary, structure, and output priorities
- be stated clearly in the instruction layer, but not repeated in the final output unless requested

### 2.1 Role assignment principle

Use the following structure internally:

```text
You are a [specific professional role].
Your task is to [specific generation task].
Output directly in [required format].
Do not explain. Do not add filler. Do not include transitions.
Keep each numbered block under 1500 Chinese characters or equivalent length.
```

### 2.2 Example roles by scenario

#### Amazon listing image set
```text
You are a professional Amazon product listing visual designer.
Your task is to generate a numbered product image set description for an Amazon listing.
Each numbered block should correspond to one image in the set.
Output directly. Do not explain. Do not add filler.
```

#### Ecommerce product visual set
```text
You are a professional ecommerce product visual art director.
Your task is to generate a structured multi-image visual plan for a product campaign.
Each numbered block should correspond to one image target.
Output directly. Do not explain. Do not add filler.
```

#### Narrative storyboard
```text
You are a professional storyboard artist and commercial film visual planner.
Your task is to generate a numbered storyboard with one shot per block.
Output directly. Do not explain. Do not add filler.
```

#### UGC ad storyboard
```text
You are a professional UGC advertising creative director.
Your task is to generate a numbered UGC-style shot list for downstream image or video generation.
Output directly. Do not explain. Do not add filler.
```

#### Reverse-engineering a video script
```text
You are a professional advertising script analyst and short-video creative strategist.
Your task is to analyze a reference video and reconstruct its structure as a reusable scene-by-scene script.
Output directly in JSON or a numbered scene format.
Do not explain. Do not add filler.
```

#### Image prompt generation
```text
You are a professional AI image prompt engineer and commercial visual director.
Your task is to generate structured prompts for downstream image generation.
Follow the dedicated image prompt best practices.
Output directly. Do not explain. Do not add filler.
```

#### Video prompt generation
```text
You are a professional AI video prompt engineer and cinematic director.
Your task is to generate structured prompts for downstream video generation.
Follow the dedicated video prompt best practices.
Output directly. Do not explain. Do not add filler.
```

---

## 3. Preferred Output Patterns

### 3.1 Numbered Block Pattern

Best for:
- image sets
- storyboards
- shot lists
- prompt batches

```text
Shot 01
[description]

Shot 02
[description]

Shot 03
[description]
```

Advantages:
- easy to split
- easy to reference
- easy to parallelize
- easy to validate

### 3.2 Labeled Multi-Field Pattern

Best for:
- structured generation
- detailed visual planning
- downstream workflows that need multi-attribute fields

```text
Shot 01
Subject:
Action:
Composition:
Lighting:
Style:
```

Advantages:
- more controllable than prose
- easier to transform into prompts
- easier to convert into JSON later

### 3.3 JSON Pattern

Best for:
- script reconstruction
- video analysis
- agent workflows
- programmatic systems

```json
{
  "scene_id": "Scene 01",
  "goal": "",
  "visual_description": "",
  "dialogue": "",
  "camera": "",
  "lighting": "",
  "style": ""
}
```

Advantages:
- machine-friendly
- easy to validate
- ideal for automation

---

## 4. Common Text-Generation Scenarios

### 4.1 Multi-Image Description Sets

Typical examples:
- Amazon listing image sets
- ecommerce carousel images
- social image series
- before/after image sets
- brand image systems

Best practice:
- assign one image per numbered block
- make each description self-contained
- clearly differentiate the function of each image
- preserve consistent product or character identity across the set
- keep each block concise

Recommended structure:

```text
Image 01
Purpose:
Subject:
Action / Scenario:
Composition:
Lighting:
Style:
Notes:

Image 02
Purpose:
Subject:
Action / Scenario:
Composition:
Lighting:
Style:
Notes:
```

For Amazon listing sets, common functions include:
- hero image
- feature callout image
- use-case image
- detail close-up
- comparison image
- packaging / included-items image
- trust / proof image

Rules:
- always number the images
- one block = one image
- no merged frames
- keep sequence logic clear

---

### 4.2 Narrative or Commercial Storyboards

Typical examples:
- ad storyboards
- short-film previsualization
- UGC video breakdowns
- concept boards
- multi-shot image generation plans

Best practice:
- each shot must have a clear number
- each shot must describe one visual moment only
- each shot should be independently usable
- maintain continuity across the sequence
- avoid mixing multiple camera events in one block

Recommended structure:

```text
Shot 01
Shot Type:
Subject:
Action:
Environment:
Lighting:
Mood:
Purpose:
```

Rules:
- every shot must be numbered
- one shot = one visual unit
- each block should remain under the length limit
- if used for image generation, every shot should be independently drawable
- if used for video generation, keep motion logic simple and direct

---

### 4.3 Reverse-Engineering a Script from a Video

Typical examples:
- analyze a reference ad and reconstruct a similar script
- infer scene structure from a viral video
- create a reusable JSON blueprint
- extract pacing, hooks, and CTA logic

Best practice:
The text model should extract:
- the overall video goal
- hook logic
- scene-by-scene structure
- likely dialogue or narration
- action beats
- editing rhythm
- CTA logic if present

Recommended JSON structure:

```json
{
  "video_goal": "",
  "target_format": "",
  "hook_type": "",
  "tone": "",
  "scenes": [
    {
      "scene_id": "Scene 01",
      "start_function": "",
      "visual_description": "",
      "dialogue_or_voiceover": "",
      "camera_or_framing": "",
      "editing_notes": ""
    },
    {
      "scene_id": "Scene 02",
      "start_function": "",
      "visual_description": "",
      "dialogue_or_voiceover": "",
      "camera_or_framing": "",
      "editing_notes": ""
    }
  ],
  "cta": ""
}
```

Why JSON is preferred:
- easier to parse
- easier to reuse
- easier to pass into Text Splitter
- easier to convert into prompts later

Rules:
- every scene must have an explicit identifier
- preserve order
- separate visual information from spoken information
- keep each scene independently interpretable

---

## 5. Prompting the Text Model for Better Outputs

### 5.1 Specify the output format explicitly

Do not ask:
- “write a storyboard”

Ask:
- “write a storyboard with numbered shots”
- “output as JSON”
- “each image description must be self-contained”
- “label each scene as Scene 01, Scene 02...”
- “do not explain, output the result directly”

### 5.2 Define the downstream use

The text model performs better when it knows what the output is for.

Examples:
- for downstream text-to-image generation
- for downstream image-to-video generation
- for Amazon listing image generation
- for Text Splitter-compatible execution
- for structured video script reconstruction

### 5.3 Ask for consistency constraints

When generating multiple items, explicitly request:
- consistent character identity
- consistent product identity
- consistent tone
- consistent visual style
- consistent story logic
- consistent output format

### 5.4 Ask for differentiation where needed

In multi-image or multi-scene outputs, also specify what should vary:
- shot type
- angle
- frame function
- emotional beat
- selling point focus
- camera distance

### 5.5 Keep units self-contained

For any numbered item, the downstream system should not need to read previous items to understand the current one.

Bad:
- “same as above but closer”

Better:
- restate the key subject / action / context clearly in each block

---

## 6. Integration with Image / Video Prompt Best Practices

When the text model is generating prompts for images or videos, it should not invent rendering logic from scratch.

Instead:
- for image prompts → follow **Image Prompt Best Practices**
- for video prompts → follow **Video Prompt Best Practices**

This document should be treated as the **text-layer orchestration guide**, while the image/video documents remain the rendering-layer guides.

Practical division:
- this document = how to structure text outputs for downstream execution
- image prompt guide = how to write strong image prompts
- video prompt guide = how to write strong video prompts

---

## 7. Copy-Ready Master Instructions

### 7.1 For multi-image sets

```text
You are a professional Amazon product listing visual designer.
Your task is to generate a numbered multi-image description set for downstream generation.
Each image must be labeled as Image 01, Image 02, etc.
Each block must be self-contained and describe exactly one image target.
Do not explain. Do not add filler. Output the result directly.
Keep each numbered block under 1500 Chinese characters or equivalent length.
```

### 7.2 For storyboards

```text
You are a professional storyboard artist and commercial film visual planner.
Your task is to generate a numbered storyboard.
Each shot must be labeled as Shot 01, Shot 02, etc.
Each shot should describe exactly one visual moment.
Do not explain. Do not add filler. Output the result directly.
Keep each numbered block under 1500 Chinese characters or equivalent length.
```

### 7.3 For reverse-engineering a reference video

```text
You are a professional advertising script analyst and short-video creative strategist.
Your task is to analyze a reference video and reconstruct a structured script in JSON.
Each scene must have a clear scene_id.
Separate visual description, dialogue/voiceover, camera notes, and scene function.
Do not explain. Do not add filler. Output the result directly.
Keep each scene block concise and reusable.
```

---

## 8. Anti-Patterns

Avoid:
- long prose with no structure
- multiple visual targets in one numbered block
- inconsistent numbering
- vague scene descriptions
- mixing narration, visual details, camera logic, and business goals into one sentence
- explanatory or conversational filler before the result

---

## 9. Final Principles

1. Structure first.
2. One block = one execution target.
3. Number everything that may need splitting.
4. Prefer JSON when the output will be reused programmatically.
5. Make every unit self-contained.
6. Generate for downstream execution, not just for readability.
7. Assign a professional role based on the scenario before generating.
8. Output directly. No explanation. No filler.
9. Keep each numbered block concise and under the length limit.
