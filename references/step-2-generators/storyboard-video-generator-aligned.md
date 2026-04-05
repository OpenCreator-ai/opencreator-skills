---
generator: 故事板分镜生视频器
node_type: videoMaker
input_modes: [text_list, image_list]
output_mode: video_list
pattern: alignment
keywords: [分镜视频, 编号对齐, 多图多段, storyboard video, alignment]
---

# 🧠 Skill：故事板分镜生视频器（Storyboard Video Generator）

## 1. Skill 定义

基于：

- 一份文本分段列表（来自 Text Splitter）
- 一份分镜图列表（与文本分段严格对应）

生成：

- 一组按镜号排列的分镜视频列表

核心能力：

> 将“分段文本结构 + 对应分镜图”扩展为“分镜级视频序列”

---

## 2. 适用场景

### 场景 1：长视频分镜生成（核心）
- 视频通常 > 15 秒
- 需要按镜头拆分并逐段生成

### 场景 2：商业 TVC / storytelling / 叙事片
- 每个镜头都有不同的参考图和剧情设定
- 需要逐镜头生成独立视频片段

### 场景 3：已有完整分镜图列表
- 已先完成 storyboard 图生成
- 需要把每张分镜图继续转成对应视频

---

## 3. 输入（Input）

### 必选输入
- Text List（必须来自 Text Splitter）
- Image List（必须为与 Text List 对应的分镜图列表）

---

## 4. 输出（Output）

```text
Video List（分镜视频列表）
```

特征：
- 每段文本对应一段视频
- 每张分镜图对应一段视频
- 数量与 Text List / Image List 一致
- 顺序严格对应镜号

---

## 5. 标准结构（Workflow）

```text
结构化脚本
↓
Text Splitter
↓
Text List
↓
故事板分镜生图器
↓
Image List
+
Text List
↓
故事板分镜生视频器
↓
Video List
```

---

## 6. 合法结构约束（最重要）

这个模块有非常强的合法输入约束。

### 6.1 文本必须来自 Text Splitter 的列表态输出

```text
[Text #1, Text #2, ..., Text #N]
```

### 6.2 图片必须是“分镜图列表”，且与文本列表严格一一对应

```text
[Image #1, Image #2, ..., Image #N]
```

### 6.3 本模块的成立前提是“三者编号对齐”

必须满足：

```text
Text #1 ↔ Image #1 ↔ Video #1
Text #2 ↔ Image #2 ↔ Video #2
...
Text #N ↔ Image #N ↔ Video #N
```

也就是说：

- 文本分段顺序
- 分镜图顺序
- 输出视频顺序

三者必须严格同编号、一一对应。

---

## 7. 为什么需要这个约束

这个模块不是“1 张参考图广播到 N 段文本”的结构。  
它和“参考图分镜生图器”不一样。

这里的目标是：

> 每一个镜头都有自己对应的那张分镜图，再结合对应文本，生成对应的视频片段。

所以它走的不是 Broadcast（广播）逻辑，而是：

```text
Alignment（编号对齐）
```

即：

```text
Text #i + Image #i → Video #i
```

---

## 8. 禁止结构（必须写清楚）

### ❌ 非法情况 1：单张 Input Image + Text List

```text
1 张参考图 + N 段文本 → 不适用于本模块
```

这种情况应走：

```text
参考图分镜生视频器
```

而不是当前这个模块。

---

### ❌ 非法情况 2：Generated Image #1 + Text List

如果图片侧只有单张生成图，即使它看起来是“一张图”，系统里也仍然是：

```text
[Image #1]
```

与 Text List 相连时，只会触发：

```text
Text #1 + Image #1 → Video #1
```

最终只生成 1 段视频，而不是 N 段。

---

### ❌ 非法情况 3：Image List 与 Text List 数量不一致

例如：

```text
Text List = 5
Image List = 4
```

这会破坏编号对齐关系，导致结构非法。

---

## 9. 编排规则（Planner 逻辑）

### 必须使用的情况
- 上游已有 Text Splitter 输出的文本列表
- 上游已有一组分镜图列表
- 需要逐镜头生成视频片段
- 每段视频都需要不同参考图

### 强制规则
```text
Text 必须来自 Text Splitter
Image 必须来自分镜图列表
Text List 数量 = Image List 数量
```

### 路径区分
```text
单图广播到多段文本 → 参考图分镜生视频器
分镜图列表逐项对应文本 → 故事板分镜生视频器（本模块）
```

---

## 10. 时长与节奏

- 每个镜头需分配时长
- 总时长 = 各镜头时长之和
- 适用于 15 秒以上的视频结构

---

## 11. 关键原则

### 原则 1：这是“编号对齐型”分镜视频生成器
它依赖的是一一对应，不是广播。

### 原则 2：文本决定镜头逻辑，图片决定镜头视觉
两者必须同时存在并且编号一致。

### 原则 3：输出是 Video List，不是最终成片
后续仍需拼接 / 剪辑 / 合成。

### 原则 4：顺序必须稳定
输出顺序 = 镜头顺序。

---

## 12. 常见错误

### ❌ 把它和“参考图分镜生视频器”混用
→ 一个是广播，一个是对齐

### ❌ 输入只有一张图
→ 不成立

### ❌ 图片列表和文本列表数量不一致
→ 不成立

### ❌ 期待它直接生成完整长视频
→ 它生成的是分镜视频列表，不是最终成片

---

## 13. 一句话总结

```text
故事板分镜生视频器 = 让文本分段列表与分镜图列表按编号一一对齐，逐镜头生成分镜视频序列的批量生成模块
```

---

## 14. 定位说明

该 Skill 是：

👉 **Text List + Image List → Video List 的编号对齐型批量生成模块**

它连接：
- 上游：Text Splitter（结构分段）+ Storyboard Image Generator（分镜图列表）
- 下游：拼接 / 剪辑 / 成片
