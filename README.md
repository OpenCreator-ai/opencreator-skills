[中文版](./README.zh-CN.md)

# OpenCreator Skills

Agent skill for operating and building OpenCreator workflows. Covers the full pipeline from template search to result delivery, and from scratch workflow design to canvas execution.

Supported scenarios: template search, template copy, workflow runs, status polling, result retrieval, workflow building, workflow editing, UGC lipsync ads, multi-shot storyboard videos, ecommerce multi-image sets, and more.

## Overview

| Field | Value |
|---|---|
| name | `opencreator-skills` |
| description | Search templates, run workflows, deliver results, or design custom workflows via the OpenCreator API |
| Product | OpenCreator |
| Environment | Production `https://api-prod.opencreator.io` |
| Entry point | [SKILL.md](./SKILL.md) |

## Installation

### Via `npx`

Install directly from the GitHub repository using the `skills` CLI:

```bash
npx skills add OpenCreator-ai/opencreator-skills
```

Install for a specific agent:

```bash
npx skills add OpenCreator-ai/opencreator-skills -a codex
```

### Via OpenClaw / ClawHub

```bash
openclaw skills install opencreator-skills
```

Or with the `clawhub` CLI:

```bash
clawhub install opencreator-skills
```

## Prerequisites

This skill requires the following environment variables:

```bash
export OPENCREATOR_API_KEY="sk_xxx"
export OPENCREATOR_BASE_URL="https://api-prod.opencreator.io"
```

- `OPENCREATOR_API_KEY` — Your OpenCreator API key
- `OPENCREATOR_BASE_URL` — Production endpoint (can also be hardcoded)

## Usage

After installation, describe your goal in natural language:

```text
Find a UGC video template and run it with this product image and description.
```

Or:

```text
Design an ecommerce multi-image workflow: input a product photo and description, output 5 images for different purposes.
```

The skill automatically selects the right mode based on the task.

## Two Modes

### Operate Mode (default)

Activated when the goal is to run an existing workflow or get quick results from a template. Most user requests follow this path.

Typical flow:

1. Search templates
2. User selects a template
3. Copy template
4. Query runtime parameters
5. Collect user inputs
6. Run workflow
7. Poll status
8. Deliver results

Key references:

- [references/api-workflows.md](./references/api-workflows.md) — Complete API guide (core)
- [references/best-practices.md](./references/best-practices.md) — Template-first strategy and design principles
- [references/operator-playbook.md](./references/operator-playbook.md) — Quick operation checklist

### Build Mode

Activated when the goal is to build a workflow graph or restructure an existing one. Only used when no suitable template exists or the user explicitly requests it.

Fixed four-step sequence:

1. Structure reverse-planning
2. Generator selection and wiring
3. Model selection and parameters
4. Prompt writing

Key references:

- [references/step-1-reverse-plan](./references/step-1-reverse-plan)
- [references/step-2-generators](./references/step-2-generators)
- [references/step-3-models](./references/step-3-models)
- [references/step-4-prompts](./references/step-4-prompts)
- [references/node-catalog.md](./references/node-catalog.md) — Node whitelist, pin rules, default models, JSON templates

## Core Concepts

### Broadcast

1 input image + N text segments = N results. Commonly used for storyboard image/video generation.

### Alignment

N images + N texts in 1:1 pairing. The i-th image must correspond to the i-th text.

### List Propagation

When `scriptSplit` outputs a text list, downstream generators auto-expand per item — no need to duplicate generator nodes.

### Shared Semantic Layer

In complex scenarios (lipsync ads, multi-branch content), generate a shared structured brief first, then fork into visual and audio branches to keep information consistent.

## Example Scenarios

### Ecommerce Multi-Image

Input a product photo and description, output 5–7 images for different purposes (hero image, feature highlights, lifestyle shots, detail close-ups, etc.).

Reference: [references/scenarios/scenario-ecommerce-multi-image.md](./references/scenarios/scenario-ecommerce-multi-image.md)

### Multi-Shot Storyboard Video

Input reference images, creative copy, or a structured script, output multiple narrative video segments.

Reference: [references/scenarios/scenario-storyboard-video.md](./references/scenarios/scenario-storyboard-video.md)

### UGC Lipsync Ad

Input product images, reference video, and product info to generate a ready-to-deploy UGC lipsync ad video.

Reference: [references/scenarios/scenario-ugc-lipsync-ad.md](./references/scenarios/scenario-ugc-lipsync-ad.md)

## Directory Structure

```text
opencreator-skills/
├── SKILL.md                    # Entry point: mode routing and flow definition
├── README.md
├── README.zh-CN.md
├── agents/
│   └── openai.yaml
└── references/
    ├── api-workflows.md        # Operate Mode core: complete API guide
    ├── best-practices.md       # Workflow design principles
    ├── node-catalog.md         # Node whitelist, pin rules, JSON templates (Build Mode core)
    ├── operator-playbook.md    # Operate quick checklist
    ├── scenarios/              # End-to-end scenario examples
    ├── step-1-reverse-plan/    # Build Step 1: structure reverse-planning
    ├── step-2-generators/      # Build Step 2: generator selection
    ├── step-3-models/          # Build Step 3: model selection
    └── step-4-prompts/         # Build Step 4: prompt writing
```

## Key Documents

| Document | Purpose |
|---|---|
| [SKILL.md](./SKILL.md) | Entry point — mode routing and execution flow |
| [references/api-workflows.md](./references/api-workflows.md) | Operate Mode core: full API pipeline guide |
| [references/node-catalog.md](./references/node-catalog.md) | Build Mode core: node whitelist, pin rules, default models, JSON templates |
| [references/best-practices.md](./references/best-practices.md) | Template-first strategy and graph design principles |
| [references/scenarios](./references/scenarios) | End-to-end examples for common scenarios |

## Guiding Principles

- Prefer template reuse over building from scratch (Operate Mode first)
- Always collect real user inputs before running
- Structure first, then implement
- Never expose `node_id`, `inputText`, or other technical fields to users — use business language
- Always re-query parameters before each run; never hardcode input node IDs
- Deliver images, videos, and audio directly as media — not just raw URLs

## Notes

- This skill targets the OpenCreator production environment
- Maintained by the OpenCreator team
- Distributable via GitHub + `npx skills add` or via ClawHub
