---

## name: opencreator-skills
description: Build OpenCreator production workflows from scratch or by editing existing graphs. Use when Codex needs to 根据用户内容创作需求在画布中设计 `nodes` 和 `edges`，执行结构反推、Generator 选择与连线、模型选型与参数填写、提示词撰写，或在完成画布设计后继续调用 OpenCreator Workflow API 执行模板搜索、运行、轮询、取结果与交付。

# Workflow Builder Skill

## 定义

本 Skill 负责在 OpenCreator 画布中，根据用户的内容创作需求，从零搭建一条完整可执行的工作流。

覆盖范围：

- 结构反推
- Generator 选择与连线
- 模型选型与参数
- 提示词撰写
- API 执行与结果交付

## 激活方式

当任务属于以下任一场景时，使用本 Skill：

- 用户要“搭一个工作流”或“设计一套 nodes + edges”
- 用户要从零实现 UGC、分镜视频、电商多图、口播广告等内容生成流程
- 用户提供了已有工作流，需要按现有结构继续修改
- 用户已经有模板或 `flow_id`，需要继续运行、轮询、获取结果

不要假设一定有“上游主脑”先分发。这个 Skill 本身就应该能独立判断何时进入搭建或执行流程。

---

## 双模式

- 优先走 Operate Mode 模式，如果没有合适的模板或者用户明确要求创建或者编辑工作流 .则进入 Build Mode

### Operate Mode

当任务重点是“搜索模板、复制模板、运行、轮询、获取结果、取消任务”时，进入 Operate Mode。

如果用户既要搭建又要执行，先走 Build Mode，再走 Operate Mode。

### Build Mode

当任务重点是“搭画布”或“改图结构”时，进入 Build Mode。

---

## 四步流程

搭建工作流时，必须按顺序执行以下四步，不可跳过。

### Step 1：结构反推与基础规格

从用户最终产物出发，逐层往前推导出抽象结构和模块依赖关系。

先回答：

- 最终产物是什么
- 是否有语义层
- 是否有视觉分支
- 是否有音频分支
- 是否需要合成层
- 所有底层输入是否都能回到用户输入或可生成的基础输入

必读文件：

- `references/step-1-reverse-plan/workflow-reverse-planner.md`
- `references/step-1-reverse-plan/node-configs.md`

输出要求：

- 宏观 Format
- Dependency Graph

### Step 2：选择 Generator 并规划连线

将 Step 1 的抽象结构落实为具体的 Generator 模块组合，并规划连线与命名。

必读文件：

- `references/step-1-reverse-plan/generator-wiring-naming-planner.md`
- `references/step-1-reverse-plan/generator-routing.md`

按需读取：

- 根据选定 Generator，继续读取 `references/step-2-generators/` 下的对应文件

### Step 3：模型选型与参数

为每个执行节点选择模型、填写 `selectedModels` 和参数。

按需读取：

- 根据涉及的节点类型，读取 `references/step-3-models/` 下的对应文件

### Step 4：撰写提示词

为需要 `inputText` 的节点编写提示词。

必读文件：

- `references/step-4-prompts/prompt-prewrite-reasoner.md`

按需读取：

- 根据节点类型，读取对应 prompt 最佳实践文件

---

## Step 2 文件路由表

根据 Step 1 反推结果中涉及的能力模块，选读对应文件。

### 文本生成类

- 用户只提供文本，需要生成结构化脚本：
  - `references/step-2-generators/reference-text-generator.md`
- 用户提供参考图片，需要提取语义或生成脚本：
  - `references/step-2-generators/reference-image-text-generator.md`
- 用户提供参考视频，需要拆解结构并仿写脚本：
  - `references/step-2-generators/reference-video-text-generator.md`
- 用户提供多种模态混合输入：
  - `references/step-2-generators/multimodal-text-generator.md`
- 需要将结构化脚本拆成分镜列表：
  - `references/step-2-generators/storyboard-text-splitter.md`

### 图像生成类

- 无参考图，纯文本生成图片：
  - `references/step-2-generators/text-to-image-generator.md`
- 有参考图，需要融合多图信息生成图片：
  - `references/step-2-generators/image-reference-generator.md`
- 分镜批量生图：
  - `references/step-2-generators/storyboard-image-generator.md`
- 图片后处理，重打光：
  - `references/step-2-generators/relight-image-generator.md`
- 图片后处理，换视角：
  - `references/step-2-generators/angle-control-image-generator.md`

### 视频生成类

- 无参考图，纯文本生成视频：
  - `references/step-2-generators/text-to-video-generator.md`
- 单张图片生成单段视频：
  - `references/step-2-generators/image-to-video-generator.md`
- 分镜视频，广播模式：
  - `references/step-2-generators/storyboard-video-generator.md`
- 分镜视频，编号对齐模式：
  - `references/step-2-generators/storyboard-video-generator-aligned.md`
- 多模态参考直接生成综合短视频：
  - `references/step-2-generators/omni-video-generator.md`
- 图片加音频生成口型视频：
  - `references/step-2-generators/lipsync-video-generator.md`
- 图片加动作参考生成动作迁移视频：
  - `references/step-2-generators/motion-transfer-video-generator.md`
- 已有视频做修改或风格迁移：
  - `references/step-2-generators/video-modify-generator.md`

### 音频生成类

- 文本生成配音：
  - `references/step-2-generators/text-to-speech-generator.md`
- 克隆音色并朗读文本：
  - `references/step-2-generators/voice-cloning-generator.md`
- 生成背景音乐：
  - `references/step-2-generators/music-generator.md`

## Step 3 文件路由表

- `textGenerator` 或 `scriptSplit`：
  - `references/step-3-models/text-generator-model-selection.md`
- `imageMaker`：
  - `references/step-3-models/text-to-image-model-selection.md`
- `imageToImage`：
  - `references/step-3-models/image-to-image-model-selection.md`
- `videoMaker`：
  - `references/step-3-models/image-to-video-model-selection.md`
- `textToVideo`：
  - `references/step-3-models/text-to-video-model-selection.md`
- `textToSpeech`：
  - `references/step-3-models/text-to-speech-model-selection.md`
- 输入节点：
  - `references/step-3-models/input-block-skill.md`

未列出的节点，优先回看：

- `references/step-1-reverse-plan/node-configs.md`

## Step 4 文件路由表

- 写任何 prompt 之前：
  - `references/step-4-prompts/prompt-prewrite-reasoner.md`
- 为 `textGenerator` 撰写结构化文本指令：
  - `references/step-4-prompts/text-prompt-best-practices.md`
- 为图像节点撰写图片 prompt：
  - `references/step-4-prompts/image-prompt-best-practices.md`
- 为视频节点撰写视频 prompt：
  - `references/step-4-prompts/video-prompt-best-practices.md`

---

## 关键概念速览

### 广播 Broadcast

1 张 Input Image 配 N 段文本，得到 N 个结果。

必须使用 `imageInput` 作为参考图来源，不能误用 Generated Image。

### 编号对齐 Alignment

N 张图配 N 段文本，第 i 张图对应第 i 段文本，数量必须一致。

### 列表态传递

当 `scriptSplit` 输出文本列表后，下游生成节点通过列表自动展开执行，不要为每段文本单独复制一个生成器。

### 共享语义层

在口播广告、多分支视频等复杂场景中，优先先产出一个共享结构化 brief，再分给视觉和音频分支，保证信息一致。

---

## 场景贯穿参考

高频场景优先读取对应端到端示例：

- UGC 口播广告：
  - `references/scenarios/scenario-ugc-lipsync-ad.md`
- 多镜头分镜视频：
  - `references/scenarios/scenario-storyboard-video.md`
- 电商多图套图：
  - `references/scenarios/scenario-ecommerce-multi-image.md`

---

## Operate Mode 补充

当图已经搭好，或用户主要是要运行与取结果时，继续读取：

- `references/api-workflows.md`
- `references/operator-playbook.md`
- `references/best-practices.md`

执行原则：

- 模板优先搜索，再决定是否从零搭建
- 运行前必须重新查询参数
- 不向用户暴露 `node_id` 等技术字段
- `inputs` 必须使用扁平结构
- `status = success` 之后再取结果
- 图片、视频、音频结果优先直接交付媒体，而不是只贴链接

---

## 输出要求

完成 Build Mode 四步流程后，最终输出标准的 `nodes` + `edges` JSON 结构。

如果当前环境有 `create_workflow` 一类工具，就直接保存；
如果没有专用工具，就通过 OpenCreator Workflow API 创建或更新工作流。

构造 `nodes` 和 `edges` 时，严格遵守：

- `references/step-1-reverse-plan/node-configs.md`
- `references/node-catalog.md`

