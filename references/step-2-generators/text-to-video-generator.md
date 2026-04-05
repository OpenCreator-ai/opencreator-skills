---
generator: 文生视频器
node_type: textToVideo
input_modes: [text]
output_mode: video
pattern: single
keywords: [文生视频, 无参考图, text to video, 纯文本, 创意预演]
---

# 文生视频器（Text to Video Generator）

## 定义

将纯文本描述直接转化为视频内容。是唯一不需要图片输入的视频生成器。

## 输入 / 输出

- **输入**：Text（视频描述，必选）
- **输出**：Video（单段）

## 适用场景

- 无任何参考图片，只有文字创意
- 创意预演 / 快速可视化想法
- 纯文本叙事短片
- 产品概念动态展示（无实物图时）

## 与 videoMaker 的关系

```text
有参考图片？
├── 有 → 使用 videoMaker（图生视频器）— 图片作为视觉锚点，控制力更强
└── 无 → 使用 textToVideo（文生视频器）— 纯靠文本描述驱动
```

> **优先级**：如果有条件生成或获取参考图片，优先走 imageMaker/imageToImage → videoMaker 路径，视觉可控性更高。textToVideo 适合"快速出原型"或"确实无图"的场景。

## 编排规则

```text
无图片 + 需要视频 → 使用该模块
有图片可用 → 优先 videoMaker
需要多镜头 → 建议先 scriptSplit 拆分，再逐段 textToVideo，或先生图再走 videoMaker 分镜路径
```

## 关键原则

- 视觉效果完全由文本描述决定，没有图片锚点
- 单段生成，不涉及列表态扩展
- 支持音画同步的模型（Sora 2、Kling 3.0、Seedance 2.0）可直接从文本渲染口型动作
- 单镜头时长建议 4-8s 最稳，超过 12s 风险上升
- Seedance 2.0 不支持真人，真人场景用 Sora 2 或 Kling

## 常见错误

- ❌ 有参考图却用 textToVideo → 失去视觉锚点控制力，应改用 videoMaker
- ❌ 一次生成长视频（>12s） → 稳定性下降，应拆分多段
- ❌ 真人场景选了 Seedance → Seedance 不支持真人
- ❌ prompt 过于抽象 → 文生视频完全依赖文本，描述应具体包含主体、动作、场景、风格
