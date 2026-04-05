# 🎬 Video Prompt Best Practices（Final Workflow-Aware Version）

## 1. 核心认知（最重要）

一个“完整单镜头描述”本质包含 4 个信息层：

```text
1. 主体（动作 + 情绪）
2. 环境（场景 / 空间）
3. 语音（台词 + 音色 + 口音）
4. 机位（Camera Angle / Framing）
5. 运镜（镜头变化）
```

👉 Video Prompt 的本质不是“描述画面”，而是：

**补齐或强化这 4 个维度中缺失的部分**

---



---

## 1.1 机位 vs 运镜（关键区分）

### 机位（Camera Angle / Framing）
- 是“静态镜头定义”
- 决定观众从哪里看

示例：
```
close-up
medium shot
wide shot
low angle
high angle
eye-level
over-the-shoulder
```

---

### 运镜（Camera Movement）
- 是“时间轴上的变化”
- 决定镜头怎么动

示例：
```
slow push-in
pan left
tracking shot
handheld movement
```

---

👉 本质区别：

```
机位 = 从哪里看
运镜 = 怎么动
```


## 2. Prompt 是否需要生成（关键判断）

在生成 Video Prompt 之前，必须判断上游已有信息：

---

### 情况 A：上游已有完整分镜脚本（最常见）

如果已经包含：

- 动作 / 情绪
- 场景
- voiceover（台词 + 音色 + 口音）
- camera（运镜）

👉 结论：

```text
❌ 不需要生成 Prompt
```

👉 直接使用脚本控制视频生成

---

### 情况 B：缺失部分信息

👉 结论：

```text
✅ 只补缺失维度
```

---

## 3. 补全逻辑（核心规则）

---

### 3.1 优先补“时间轴相关信息”

必须优先补：

```text
- 动作变化（Motion）
- 情绪变化（Emotion）
- 运镜变化（Camera Movement）
```

因为这些是视频生成的核心控制变量

---

### 3.2 语音缺失（必须补）

如果缺少 voiceover：

必须补：

```text
Voiceover:
Content: "完整台词"
Voice: [音色]
Accent: [口音]
```

👉 否则：

- lip-sync 不稳定
- 语音不一致

---

### 3.3 有图片输入时（关键优化）

如果存在 Image 输入：

```text
❌ 不要描述：
- 人物外貌
- 场景环境

✅ 只描述：
- 动作
- 情绪
- 运镜
```

---

### 3.4 无图片输入时

必须完整描述：

```text
主体 + 环境 + 动作 + 运镜 + 情绪
```

---

## 4. Prompt 生成结构（最终规范）

---

### 最小结构（推荐）

```text
[Camera Angle]
+ [Camera Movement]
+ [Motion]
+ [Emotion]
```

---

### 完整结构（无上游信息时）

```text
Camera
+ Motion
+ Emotion
+ Environment
+ Style
```

---

### 示例（有图片输入）

```text
slow push-in,
leans forward slightly, pauses,
expression shifts from neutral to confident
```

---

### 示例（无图片输入）

```text
medium shot, slow push-in,
a young woman speaks confidently, slight hand gesture,
warm indoor lighting,
cinematic style
```

---

## 5. 优先级规则

```text
Script > Image > Prompt
```

👉 Prompt 永远是补充层，不是主控制层

---

## 6. 核心原则

### 原则 1：Prompt 只负责“动态控制”

不负责：

- 描述主体
- 描述环境（如果已有）

---

### 原则 2：不要重复已有信息

重复会导致：

- 结果不稳定
- 模型混淆

---

### 原则 3：动作必须可执行

❌ excited  
✅ leans forward, raises voice

---

### 原则 4：保持极简

```text
1 motion + 1 camera + 1 emotion
```

---

## 7. 常见错误

### ❌ 重写整个脚本
→ 破坏结构

---

### ❌ 描述人物外貌（已有图片）
→ 冗余且不稳定

---

### ❌ 忽略 voiceover
→ 语音不可控

---

### ❌ 写成长段落
→ 不可执行

---

## 8. 一句话总结

```text
Video Prompt = 补充“时间轴上的变化”（动作 / 情绪 / 运镜），而不是重写内容
```

---

## 9. 本质理解（升级）

👉 Prompt 不再是描述层  
👉 而是：

**执行控制层（Execution Control Layer）**
