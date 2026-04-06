---
name: opencreator-skills
description: >-
  Operate and build OpenCreator workflows via API. Use when the user wants to
  search templates, run workflows, poll results, deliver generated media, or
  design custom workflow graphs with nodes and edges on OpenCreator. Triggers:
  opencreator, workflow, template, UGC, ecommerce images, storyboard video,
  lipsync, content creation pipeline.
---

# OpenCreator Workflow Skill

## Activation

Use this skill when the task involves any of:

- Searching or running OpenCreator templates
- Running an existing workflow and getting results
- Building or editing a workflow graph (nodes + edges)
- UGC, storyboard video, ecommerce multi-image, or similar content creation

## Mode Decision

```
User request
  │
  ├─ Run template / get results / "帮我做 XX" ──► Operate Mode (default)
  │
  └─ Create workflow / edit graph / "从零搭" / no suitable template ──► Build Mode
```

**Always try Operate Mode first.** Switch to Build Mode only when:

- No suitable template exists after searching
- The user explicitly asks to create or edit a workflow
- The required graph differs materially from any available template

If a task needs both, do Build first (produce the graph), then Operate (run it).

---

## Operate Mode

**Must read:** `references/api-workflows.md`

This single file covers the complete Operate flow:

1. Configuration (Base URL, API Key)
2. Search templates by keyword
3. Present candidates, user selects
4. Copy template → get `flow_id`
5. Query runtime parameters
6. Collect user inputs (ask every field, never use defaults)
7. Run workflow
8. Poll status
9. Deliver results (media directly, not just links)

Supplementary (read only when you need deeper tactics):

- `references/best-practices.md` — template-first strategy and design principles

### Operate Hard Rules

- Always copy template before running (public templates are read-only)
- Always query parameters before each run (node IDs can change)
- `inputs` must be flat: `{ "node_id": "value" }` — never wrap in extra object
- Never expose `node_id` / `inputText` / `imageBase64` to users — use business language
- **Search results must be ranked by relevance and only show top 5 to the user**
- **After starting a run, you MUST poll until terminal state (success/failed/cancelled) — never stop and wait for the user to ask. This is your #1 obligation.**
- **On success, immediately fetch results and deliver media to the user — do not end your turn without delivering.**
- Poll every 10 s for text/image, 30 s for video
- Deliver media directly, not just URLs

---

## Build Mode

When building or editing a workflow graph, follow these four steps in order. Do not skip any step.

### Step 1: Structure Reverse-Planning

Work backward from the user's final deliverable to identify the abstract structure and module dependencies.

Answer these questions first:

- What is the final output?
- Does it need a semantic layer (text/script generation)?
- Does it need a visual branch (image/video)?
- Does it need an audio branch (TTS/music)?
- Does it need a compositing layer?
- Can all leaf inputs trace back to user input or generatable primitives?

Must read:

- `references/step-1-reverse-plan/workflow-reverse-planner.md`
- `references/node-catalog.md`

Output: Macro Format + Dependency Graph

### Step 2: Generator Selection & Wiring

Map abstract modules to concrete generators and plan edges + naming.

Must read:

- `references/step-1-reverse-plan/generator-wiring-naming-planner.md`
- `references/step-1-reverse-plan/generator-routing.md`

Then read the matching file in `references/step-2-generators/` (see routing table below).

### Step 3: Model Selection & Parameters

Choose models, fill `selectedModels` and parameters for each node.

Hard rule before choosing any model:

- Treat the `Confirmed model IDs` tables in each Step 3 file as the source of truth for model IDs
- Only use model IDs that are explicitly mapped to the current node type / atom
- If a model is recommended in prose but its exact ID is not listed for that atom, do not use it
- Never translate a marketing name (for example `Sora 2`, `GPT Image 1.5`, `Seedream 5.0 Lite`) into a guessed model ID
- If an atom has no dedicated Step 3 file, use `references/node-catalog.md` as the fallback source of truth
- If an atom still has no stable model-selection entry after checking those docs, keep the documented fixed behavior and do not invent `selectedModels`

Read the matching file in `references/step-3-models/` (see routing table below).

### Step 4: Prompt Writing

Write prompts for nodes that need `inputText`.

Must read:

- `references/step-4-prompts/prompt-prewrite-reasoner.md`

Then read the matching prompt best-practices file (see routing table below).

---

## Step 2 Generator Routing Table

### Text Generation

- Text only → script: `references/step-2-generators/reference-text-generator.md`
- Reference image → text: `references/step-2-generators/reference-image-text-generator.md`
- Reference video → text: `references/step-2-generators/reference-video-text-generator.md`
- Multimodal input → text: `references/step-2-generators/multimodal-text-generator.md`
- Script → storyboard split: `references/step-2-generators/storyboard-text-splitter.md`

### Image Generation

- Text → image: `references/step-2-generators/text-to-image-generator.md`
- Multi-image reference → image: `references/step-2-generators/image-reference-generator.md`
- Storyboard batch images: `references/step-2-generators/storyboard-image-generator.md`
- Relight: `references/step-2-generators/relight-image-generator.md`
- Angle control: `references/step-2-generators/angle-control-image-generator.md`

### Video Generation

- Text → video: `references/step-2-generators/text-to-video-generator.md`
- Image → video: `references/step-2-generators/image-to-video-generator.md`
- Storyboard broadcast: `references/step-2-generators/storyboard-video-generator.md`
- Storyboard aligned: `references/step-2-generators/storyboard-video-generator-aligned.md`
- Omni video: `references/step-2-generators/omni-video-generator.md`
- Lipsync: `references/step-2-generators/lipsync-video-generator.md`
- Motion transfer: `references/step-2-generators/motion-transfer-video-generator.md`
- Video modify / style transfer: `references/step-2-generators/video-modify-generator.md`

### Audio Generation

- Text → speech: `references/step-2-generators/text-to-speech-generator.md`
- Voice cloning: `references/step-2-generators/voice-cloning-generator.md`
- Music: `references/step-2-generators/music-generator.md`

## Step 3 Model Routing Table

- Source of truth for exact IDs: the `Confirmed model IDs` table in each file below, with `references/node-catalog.md` as fallback for nodes without a dedicated Step 3 file
- `textGenerator` / `scriptSplit`: `references/step-3-models/text-generator-model-selection.md`
- `imageMaker`: `references/step-3-models/text-to-image-model-selection.md`
- `imageToImage`: `references/step-3-models/image-to-image-model-selection.md`
- `videoMaker`: `references/step-3-models/image-to-video-model-selection.md`
- `textToVideo`: `references/step-3-models/text-to-video-model-selection.md`
- `textToSpeech`: `references/step-3-models/text-to-speech-model-selection.md`
- Input nodes: `references/step-3-models/input-block-skill.md`
- Not listed above: check `references/node-catalog.md`

## Step 4 Prompt Routing Table

- Before writing any prompt: `references/step-4-prompts/prompt-prewrite-reasoner.md`
- `textGenerator` prompts: `references/step-4-prompts/text-prompt-best-practices.md`
- Image node prompts: `references/step-4-prompts/image-prompt-best-practices.md`
- Video node prompts: `references/step-4-prompts/video-prompt-best-practices.md`

---

## Key Concepts

### Broadcast

1 image + N texts → N results. Must use `imageInput` as the reference image source, not a generated image.

### Alignment

N images + N texts, 1:1 pairing. Counts must match exactly.

### List Propagation

`scriptSplit` outputs a text list; downstream generators auto-expand per item — do not duplicate generator nodes.

### Shared Semantic Layer

In complex scenarios (lipsync ads, multi-branch video), generate a shared structured brief first, then fork to visual and audio branches.

---

## Scenario References

- UGC lipsync ad: `references/scenarios/scenario-ugc-lipsync-ad.md`
- Storyboard video: `references/scenarios/scenario-storyboard-video.md`
- Ecommerce multi-image: `references/scenarios/scenario-ecommerce-multi-image.md`

---

## Build Mode Output

After completing the four steps, output standard `nodes` + `edges` JSON.

Node and edge schema: `references/node-catalog.md`

Save via `create_workflow` tool if available, otherwise via the Workflow PATCH API (see `references/api-workflows.md` §10).
