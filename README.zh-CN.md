[English](./README.md)

# OpenCreator Skills

OpenCreator 工作流技能。通过 OpenCreator 的工作流能力，完成从模板搜索到结果交付、从零搭建到画布设计的全链路工作。

覆盖场景：模板搜索、模板复制、工作流运行、运行状态查询、结果获取、工作流搭建、工作流编辑，以及 UGC 口播广告、多镜头分镜视频、电商多图套图等内容生产场景。

## 基本信息

| 字段 | 内容 |
|---|---|
| name | `opencreator-skills` |
| description | 通过 OpenCreator API 搜索模板、运行工作流、交付结果，或从零设计自定义工作流 |
| 适用产品 | OpenCreator |
| 适用环境 | 正式环境 `https://api-prod.opencreator.io` |
| 技能入口 | [SKILL.md](./SKILL.md) |

## 安装

### 通过 `npx` 安装

如果你希望直接从 GitHub 仓库安装，推荐使用 `skills` CLI：

```bash
npx skills add OpenCreator-ai/opencreator-skills
```

安装到指定 Agent：

```bash
npx skills add OpenCreator-ai/opencreator-skills -a codex
```

### 通过 OpenClaw / ClawHub 安装

```bash
openclaw skills install opencreator-skills
```

如果使用 `clawhub` CLI，也可以：

```bash
clawhub install opencreator-skills
```

## 使用前提

使用这份 skill 时，默认依赖以下环境变量：

```bash
export OPENCREATOR_API_KEY="sk_xxx"
export OPENCREATOR_BASE_URL="https://api-prod.opencreator.io"
```

说明：

- `OPENCREATOR_API_KEY`：OpenCreator API Key
- `OPENCREATOR_BASE_URL`：默认正式环境地址，不改也可以直接写死为正式环境

## 使用

安装后，在 Codex 中直接描述你的目标即可。

例如：

```text
帮我找一个 UGC 视频模板，跑一下这个产品图和产品描述。
```

或：

```text
帮我设计一个电商多图工作流，输入商品图和产品描述，输出 5 张不同用途的商品图。
```

这份 skill 会根据任务类型自动进入不同模式。

## 双模式

### Operate Mode（默认）

当任务目标是"运行已有工作流"或"通过模板快速出结果"时，进入 Operate Mode。大多数用户请求走这个路径。

典型流程：

1. 搜索模板
2. 用户选择模板
3. 复制模板
4. 查询运行参数
5. 补齐用户输入
6. 运行 workflow
7. 轮询状态
8. 获取结果并交付

关键资料：

- [references/api-workflows.md](./references/api-workflows.md) — 完整 API 调用指南（核心）
- [references/best-practices.md](./references/best-practices.md) — 模板优先策略与设计原则
- [references/operator-playbook.md](./references/operator-playbook.md) — 操作快速检查清单

### Build Mode

当任务目标是"搭建工作流"或"重构工作流结构"时，进入 Build Mode。仅在没有合适模板或用户明确要求时使用。

执行顺序固定为四步：

1. 结构反推
2. 选择 Generator 并规划连线
3. 模型选型与参数
4. 撰写提示词

关键资料：

- [references/step-1-reverse-plan](./references/step-1-reverse-plan)
- [references/step-2-generators](./references/step-2-generators)
- [references/step-3-models](./references/step-3-models)
- [references/step-4-prompts](./references/step-4-prompts)
- [references/node-catalog.md](./references/node-catalog.md) — 节点白名单、Pin 规则、默认模型、JSON 模板

## 核心概念

### Broadcast

1 张 Input Image 对应 N 段文本，生成 N 个结果。
常用于分镜生图、分镜生视频。

### Alignment

N 张图对应 N 段文本，第 i 张图必须对应第 i 段文本。
常用于分镜图和分镜视频一一配对。

### 列表态传递

当 `scriptSplit` 输出文本列表后，下游生成节点按列表自动展开执行，不需要手工复制多个相同节点。

### 共享语义层

在口播广告、多分支内容生成等复杂场景中，优先先生成一个共享结构化 brief，再分给视觉和音频分支，保证信息一致。

## 典型场景

### 电商多图套图

输入商品图和商品描述，输出 5 到 7 张不同用途的商品图，例如主图、卖点图、场景图、细节图等。

参考：[references/scenarios/scenario-ecommerce-multi-image.md](./references/scenarios/scenario-ecommerce-multi-image.md)

### 多镜头分镜视频

输入参考图、创意文案或结构化脚本，输出多段叙事视频。

参考：[references/scenarios/scenario-storyboard-video.md](./references/scenarios/scenario-storyboard-video.md)

### UGC 口播广告

输入产品图、参考视频、产品信息等素材，生成可直接投放的 UGC 口播广告视频。

参考：[references/scenarios/scenario-ugc-lipsync-ad.md](./references/scenarios/scenario-ugc-lipsync-ad.md)

## 目录结构

```text
opencreator-skills/
├── SKILL.md                    # 技能入口，模式路由与流程定义
├── README.md                   # 英文版说明
├── README.zh-CN.md             # 中文版说明
├── agents/
│   └── openai.yaml
└── references/
    ├── api-workflows.md        # Operate Mode 核心：完整 API 调用指南
    ├── best-practices.md       # 工作流设计原则
    ├── node-catalog.md         # 节点白名单、Pin 规则、JSON 模板（Build Mode 核心参考）
    ├── operator-playbook.md    # Operate 快速检查清单
    ├── scenarios/              # 端到端场景示例
    ├── step-1-reverse-plan/    # Build Step 1：结构反推
    ├── step-2-generators/      # Build Step 2：Generator 选择
    ├── step-3-models/          # Build Step 3：模型选型
    └── step-4-prompts/         # Build Step 4：提示词撰写
```

## 关键文档

| 文档 | 作用 |
|---|---|
| [SKILL.md](./SKILL.md) | 主入口，定义模式路由与执行流程 |
| [references/api-workflows.md](./references/api-workflows.md) | Operate Mode 核心：API 全链路调用手册 |
| [references/node-catalog.md](./references/node-catalog.md) | Build Mode 核心：节点白名单、Pin 规则、默认模型、JSON 模板 |
| [references/best-practices.md](./references/best-practices.md) | 模板优先策略与图设计原则 |
| [references/scenarios](./references/scenarios) | 高频业务场景的端到端示例 |

## 执行原则

- 先模板复用，后从零搭建（Operate Mode 优先）
- 先真实用户输入，后运行
- 先结构，后实现
- 不向用户暴露 `node_id`、`inputText` 等技术字段
- 运行前必须重新查询参数，不硬编码运行输入
- 图片、视频、音频结果优先直接交付媒体，而不是只贴链接

## 说明

- 这份 skill 以 OpenCreator 正式环境为准
- 这份 skill 由我们自己维护
- 既可以通过 GitHub + `npx skills add` 分发，也可以发布到 ClawHub
