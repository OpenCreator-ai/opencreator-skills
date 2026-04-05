---
generator: 故事板分镜生图器
node_type: imageToImage
input_modes: [text_list, image_input]
output_mode: image_list
pattern: broadcast
keywords: [分镜图, 批量生图, 广播, storyboard, 列表态]
---

# 🧠 Skill：故事板分镜生图器（Storyboard Image Generator）

## 1. Skill 定义

基于：

- 一份文本分段列表（来自 Text Splitter）
- 一张或多张输入型参考图片（Image Input）

生成：

- 一组按镜号排列的分镜图列表

核心能力：

> 将“分段文本结构”在统一视觉参考下扩展为“分镜图序列”

---

## 2. 适用场景（核心高频场景）

该模块主要用于：

### 场景 1：分镜图生成（最核心）

- 已有结构化脚本
- 已通过 Text Splitter 拆分为分段文本
- 需要生成一组镜头图

---

### 场景 2：长视频视觉预演

- 视频 > 30s
- 需要先生成 storyboard
- 每段文本 → 一张图

---

### 场景 3：统一角色 / 场景的镜头序列

- 固定人物（avatar）
- 固定场景
- 多镜头变化

---

## 3. 输入（Input）

### 必选输入

- Text List（必须来自 Text Splitter）
- Image（必须为 Image Input）

---

### 输入约束（非常重要）

#### 约束 1：文本必须是列表态

```text
Text Splitter 输出：
[Text #1, Text #2, ..., Text #N]
```

---

#### 约束 2：图片必须是 Input Image（非生成结果）

允许：

```text
Image Input（用户上传 / 外部输入）
```

禁止：

```text
Generated Image（来自生成器的结果）
```

---

## 4. 输出（Output）

```text
Image List（分镜图列表）
```

特征：

- 每段文本对应一张图
- 数量与 Text List 一致
- 顺序严格对应镜号

---

## 5. 标准结构（Workflow）

```text
结构化脚本
↓
Text Splitter
↓
Text List
+
Image Input
↓
故事板分镜生图器
↓
Image List
```

---

## 6. 核心机制（最重要部分）

### 6.1 广播机制（Broadcast）

该模块的本质执行逻辑是：

```text
1 张参考图 × N 段文本 → N 张图
```

也就是：

- 图片作为“全局参考锚点”
- 被广播到每一段文本上

---

### 6.2 而不是编号对齐（Alignment）

系统默认行为是：

```text
Text #i ↔ Image #i
```

但该模块需要的是：

```text
Image（单一） → 广播给所有 Text
```

---

## 7. 为什么必须使用 Input Image（关键原理）

### 7.1 生成式图片本质是列表结构

即使只生成一张图：

```text
Generated Image = [Image #1]
```

---

### 7.2 与 Text List 连接时会触发编号对齐

```text
Text List = [Text #1, Text #2, ..., Text #N]
Image List = [Image #1]
```

系统执行：

```text
Text #1 + Image #1 → Output #1
```

而不会扩展为 N 份。

---

### 7.3 导致结果退化为单图输出

最终结果：

```text
只有 1 张图
```

而不是：

```text
N 张分镜图
```

---

### 7.4 正确做法：使用 Input Image

Input Image 不参与编号对齐，而是：

👉 作为全局参考，被广播到所有文本分段

---

## 8. 编排规则（Planner 使用逻辑）

### 必须使用的情况

- 上游为 Text Splitter（文本列表）
- 需要生成分镜图序列
- 需要统一视觉参考

---

### 强制规则

```text
Text 必须来自 Text Splitter
Image 必须来自 Image Input
```

---

### 禁止结构

```text
Generated Image + Text List → ❌ 错误
```

---

## 9. 关键原则

### 原则 1：这是“列表扩展器”，不是普通生图器

核心价值：

- 将单次生成扩展为分镜级批量生成

---

### 原则 2：文本驱动镜头数量

- 有多少段文本 → 生成多少张图

---

### 原则 3：图片是“全局锚点”

- 控制人物一致性 / 场景一致性

---

### 原则 4：顺序必须稳定

- 输出顺序 = 镜头顺序

---

## 10. 常见错误

### ❌ 使用生成图片作为参考
→ 会导致只能输出 1 张图

### ❌ 不使用 Text Splitter
→ 无法进入列表态

### ❌ 误认为是普通 image-to-image
→ 实际是批量结构生成模块

---

## 11. 一句话总结

```text
故事板分镜生图器 = 用 1 张输入参考图，将文本分段列表扩展为一组分镜图序列
```

---

## 12. 定位说明

该 Skill 是：

👉 **Text List → Image List 的关键批量生成模块**

它连接：

- 上游：Text Splitter（结构分段）
- 输入：Image Input（视觉锚点）
- 下游：视频生成 / 拼接 / 剪辑
