---
generator: 参考图生文器
node_type: textGenerator
input_modes: [text, image]
output_mode: text
pattern: single
keywords: [参考图, 产品图, 卖点分析, 商品, 视觉语义, image reference]
---

# 🧠 Skill：参考图生文器（Reference Image → Structured Text Generator）

## 1. Skill 定义

基于输入的：

- 文本需求（可选）
- 单张或多张参考图片

生成：

- **结构化文本脚本（JSON）**
- **产品卖点分析**

核心能力：

> 从“视觉信息”中提取语义，并转化为可用于生成或营销的结构化文本

---

## 2. 核心能力（两大主场景）

### 场景 1：参考图 → 结构化描述

用于：

- 爆款内容理解
- 视频结构拆解前置
- 风格分析
- 视觉语义提取

输出：

- 场景描述
- 主体信息
- 构图要素
- 风格标签

---

### 场景 2：商品图 → 卖点提炼 / 营销脚本

用于：

- 电商广告
- UGC脚本生成
- 商品理解
- listing内容生成

输出：

- 产品卖点
- 用户痛点
- 使用场景
- 口播脚本

---

## 3. 输入（Input）

### 必选输入

- image（单张或多张）

### 可选输入

- 用户文本需求（如：生成广告脚本）

---

## 4. 输出（Output）

必须为结构化输出（JSON）

### 场景 A：视觉理解

```json
{
  "style": "",
  "scene": "",
  "subject": "",
  "composition": "",
  "lighting": "",
  "camera": ""
}
```

---

### 场景 B：商品分析

```json
{
  "product_type": "",
  "key_features": [],
  "selling_points": [],
  "target_users": [],
  "use_cases": [],
  "script": ""
}
```

---

## 5. 标准结构（Workflow 位置）

```text
Text (需求，可选)
+
Image（参考图 / 商品图）
↓
参考图生文器
↓
结构化文本（JSON）
```

---

## 6. 编排规则（Planner 使用逻辑）

### 必须使用的情况

- 用户提供参考图
- 需要理解视觉内容
- 需要生成脚本但缺少语义输入

---

### 决策逻辑

```text
有参考图 → 必走该模块
无参考图 → 不使用
```

---

### 分支逻辑

```text
图像偏风格 → 输出结构化视觉描述
图像偏商品 → 输出卖点 + 脚本
图像混合 → 两者都输出
```

---

## 7. 模型建议

### 推荐：

- GPT-4o / GPT-4o mini（多模态理解强）

### 降级：

- 纯文本模型（效果较弱）

---

## 8. 常见错误

### ❌ 错误 1：只描述图像，不结构化
→ 必须 JSON 输出

### ❌ 错误 2：输出过于抽象
→ 必须具体可用

### ❌ 错误 3：忽略商品信息
→ 必须提取卖点 / 使用场景

### ❌ 错误 4：把它当 caption 工具
→ 它是“生成输入”，不是“描述工具”

---

## 9. 示例

### 输入：

- 商品图（口红）
- 文本：“生成TikTok广告脚本”

### 输出：

```json
{
  "product_type": "lipstick",
  "key_features": ["matte finish", "long-lasting"],
  "selling_points": ["non-drying", "high pigment"],
  "target_users": ["young women"],
  "use_cases": ["daily makeup", "dating"],
  "script": "This lipstick stays on all day..."
}
```

---

## 10. 一句话总结

```text
参考图生文器 = 把视觉信息转成可用于生成或营销的结构化文本
```

---

## 11. 定位说明

该 skill 是：

👉 **视觉 → 语义 的桥梁模块**

它连接：

- 上游：用户输入 / 图片
- 下游：脚本生成 / 视频生成 / 内容结构推理
