# 🖼️ Image Prompt Best Practices（V3 · Workflow-Aware Version）

## 0. 核心认知（最重要）

一个“完整图像描述”应包含以下 7 个维度：

```text
1. 主体外貌（Subject Appearance）
2. 环境（Environment）
3. 动作 / 表情（Action & Expression）
4. 景别（Shot Type）
5. 机位（Camera Angle）
6. 构图（Composition）
7. 灯光（Lighting）
8. 视觉风格（Style）
```

👉 Image Prompt 的本质：

**补齐这些维度中缺失的信息，而不是重复已有信息**

---

## 1. Prompt 是否需要补全（关键判断）

在生成 Image Prompt 之前，必须判断上游输入：

---

### 情况 A：上游已有完整描述

如果已经包含：

- 主体外貌
- 环境
- 动作
- 构图 / 机位 / 景别
- 灯光 / 风格

👉 结论：

```text
❌ 不需要补充 Prompt
```

👉 直接进入图像生成

---

### 情况 B：部分信息缺失

👉 结论：

```text
✅ 只补缺失维度
```

---

### 情况 C：已有图片参考（非常关键）

如果存在 Image 输入：

👉 结论：

```text
❌ 不要描述主体外貌
❌ 不要描述环境
```

👉 只补：

```text
动作 / 表情
构图 / 景别 / 机位
灯光
风格
```

---

## 2. Prompt 生成策略（核心规则）

---

### 2.1 优先补“视觉控制维度”

优先级：

```text
构图 / 景别 / 机位 > 灯光 > 风格 > 动作
```

👉 因为这些直接影响画面质量与稳定性

---

### 2.2 有图片输入时（重要）

只写：

```text
Camera Angle（机位）
Shot Type（景别）
Composition（构图）
Lighting（灯光）
Style（风格）
+ 可选动作
```

---

### 2.3 无图片输入时

必须完整描述：

```text
主体 + 环境 + 动作 + 构图 + 灯光 + 风格
```

---

## 3. Prompt 结构（标准）

---

### 最小结构（有图输入）

```text
Shot Type
+ Camera Angle
+ Composition
+ Lighting
+ Style
```

---

### 完整结构（无图输入）

```text
Subject
+ Environment
+ Action / Expression
+ Shot Type
+ Camera Angle
+ Composition
+ Lighting
+ Style
```

---

## 4. 模块说明（关键区分）

---

### 4.1 景别（Shot Type）

```text
close-up
medium shot
wide shot
macro shot
```

👉 决定“画面范围”

---

### 4.2 机位（Camera Angle）

```text
eye-level
low angle
high angle
over-the-shoulder
top-down
```

👉 决定“观看角度”

---

### 4.3 构图（Composition）

```text
rule of thirds
centered composition
symmetrical composition
off-center framing
```

👉 决定“视觉结构”

---

### 4.4 灯光（Lighting）

```text
soft natural daylight
golden hour lighting
studio lighting
low-key lighting
```

---

### 4.5 风格（Style）

```text
photorealistic
cinematic
UGC style
fashion editorial
minimalist
```

---

## 5. 示例

---

### 示例 1（有图片输入）

```text
close-up shot,
eye-level angle,
rule of thirds composition,
soft natural lighting,
cinematic photorealistic style
```

---

### 示例 2（无图片输入）

```text
a young woman smiling confidently,
in a modern indoor workspace,
slight head tilt and relaxed posture,
medium shot,
eye-level angle,
balanced composition,
soft daylight,
UGC realistic style
```

---

## 6. 核心原则

---

### 原则 1：不要重复已有信息

- 已有主体 → 不写主体
- 已有环境 → 不写环境

---

### 原则 2：优先控制“画面结构”

👉 构图 / 机位 / 景别 = 最核心

---

### 原则 3：避免抽象描述

❌ beautiful  
✅ soft lighting, natural skin texture

---

### 原则 4：模块化表达

```text
Shot + Angle + Composition + Lighting + Style
```

---

## 7. 常见错误

---

### ❌ 重复描述图片内容
→ 降低稳定性

---

### ❌ 缺少构图 / 机位
→ 画面不可控

---

### ❌ 写成长段自然语言
→ 难以解析

---

### ❌ 忽略风格
→ 输出随机

---

## 8. 一句话总结

```text
Image Prompt = 补齐“视觉结构与风格控制”，而不是描述内容
```

---

## 9. 本质理解

👉 Image Prompt ≠ 描述图像  
👉 Image Prompt =

**控制画面结构与视觉呈现的执行层**
