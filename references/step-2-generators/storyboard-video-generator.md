---
generator: 参考图分镜生视频器
node_type: videoMaker
input_modes: [text_list, image_input]
output_mode: video_list
pattern: broadcast
keywords: [分镜视频, 广播, 单图多段, storyboard video, broadcast]
---

# 🧠 Skill：参考图分镜生视频器（Storyboard Video Generator）

## 1. Skill 定义

基于：

- 一份文本分段列表（来自 Text Splitter）
- 一张参考图片（Image Input，作为统一主体/元素锚点）

生成：

- 一组按镜号排列的视频列表（分镜视频）

核心能力：

> 将“分段文本结构”在统一视觉参考下扩展为“分镜级视频序列”

---

## 2. 适用场景

### 场景 1：长视频分镜生成（核心）
- 视频通常 > 15 秒
- 需要按镜头拆分并逐段生成

### 场景 2：统一主体的连续镜头
- 固定人物 / 商品 / 场景
- 多镜头变化但保持一致性

### 场景 3：广告 / UGC 批量视频生产
- 结构化脚本 → 分镜 → 批量视频

---

## 3. 输入（Input）

### 必选输入
- Text List（必须来自 Text Splitter）
- Image（必须为 Image Input）

### 输入约束（关键）
- 文本必须是列表态（分段/分镜）
- 图片必须为输入型 Image Input（不可为生成式图片结果）

---

## 4. 输出（Output）

```text
Video List（分镜视频列表）
```

特征：
- 每段文本对应一段视频
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
Image Input（统一主体）
↓
参考图分镜生视频器
↓
Video List
```

---

## 6. 核心机制（重点）

### 6.1 列表扩展（List Expansion）
```text
Text List（N 段） → Video List（N 段）
```

### 6.2 参考图广播（Broadcast）
```text
1 张参考图 × N 段文本 → N 段视频
```

参考图作为“全局锚点”，被应用到每一段视频生成中，用于保持主体一致性。

### 6.3 文本驱动镜头
- 每一段文本定义一个镜头的内容与时长
- 最终视频总时长 = 各镜头时长之和

---

## 7. 为什么必须使用 Image Input（关键原理）

生成式图片结果本质为列表结构（即使只有一张）：
```text
Generated Image = [Image #1]
```

与 Text List 连接会触发编号对齐（Alignment）：
```text
Text #1 ↔ Image #1 → 仅生成 1 段视频
```

而非所需的广播（Broadcast）。

因此必须使用：
```text
Image Input（不参与编号对齐，可被广播）
```

---

## 8. 编排规则（Planner 逻辑）

### 必须使用的情况
- 上游为 Text Splitter（文本列表）
- 目标为多镜头视频
- 需要统一主体/元素

### 强制规则
```text
Text 必须来自 Text Splitter
Image 必须来自 Image Input
```

### 禁止结构
```text
Generated Image + Text List → ❌（会退化为单视频）
```

---

## 9. 时长与节奏

- 每个镜头需分配时长（由上游结构或模型参数控制）
- 总时长由各镜头累加得到
- 适用于 15 秒以上的视频结构

---

## 10. 关键原则

### 原则 1：这是“分镜级视频生成器”
- 非单段视频生成
- 面向多镜头序列

### 原则 2：文本决定数量
- N 段文本 → N 段视频

### 原则 3：图片是统一锚点
- 保持人物/商品/风格一致

### 原则 4：顺序稳定
- 输出顺序 = 镜头顺序

---

## 11. 常见错误

### ❌ 使用生成图片作为参考
→ 会导致只生成 1 段视频

### ❌ 未使用 Text Splitter
→ 无法进入列表态

### ❌ 期待单模块完成长视频
→ 应使用该分镜模块而非单段视频模块

---

## 12. 一句话总结

```text
参考图分镜生视频器 = 用 1 张输入参考图，将文本分段列表扩展为一组分镜视频序列
```

---

## 13. 定位说明

该 Skill 是：

👉 **Text List → Video List 的关键批量生成模块**

它连接：
- 上游：Text Splitter（结构分段）
- 输入：Image Input（统一主体）
- 下游：拼接 / 剪辑 / 成片
