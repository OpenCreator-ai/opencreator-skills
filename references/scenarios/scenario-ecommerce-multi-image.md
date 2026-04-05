# 场景贯穿：电商多图套图（亚马逊 Listing 风格）

> 本文件展示一个完整场景从 Step 1 到 Step 4 的全链路。适用于：用户有一张产品图 + 产品描述，想生成一组用途不同的商品图（主图、卖点图、场景图、细节图等）。

---

## Step 1：结构反推

### 最终产物

多张商品图（5-7 张，各有不同用途：主图、卖点、场景、细节等）

### 反推过程

```text
多张商品图
→ 需要：每张图一个独立 prompt
→ 多个独立 prompt 从哪来？→ 需要 scriptSplit 拆分 ← 结构化多图描述集
→ 结构化描述集从哪来？→ 需要生文器 ← 产品图 + 产品描述
→ 产品图和描述从哪来？→ 用户上传/输入
```

### 抽象结构

```text
输入层（产品图 + 产品描述）
↓
语义层（分析产品 → 生成编号多图描述集 Image 01/02/03…）
↓
分镜结构层（scriptSplit → 文本列表）
↓
视觉分支（1 张产品参考图 × N 条描述 → N 张商品图）[广播]
↓
输出（图片列表）
```

### 追问检查

- 用户提供了产品图吗？→ 没有则追问
- 用户提供了产品描述/URL吗？→ 没有则追问
- 想要几张图？什么用途？→ 影响 A 的角色设定

---

## Step 2：选择 Generator 并连线

| 抽象模块 | Generator | node.type | 说明 |
|----------|-----------|-----------|------|
| 语义层 | 参考图生文器 A | `textGenerator` | 接 imageInput + textInput，输出编号多图描述集 |
| 分镜结构层 | 故事板分文器 B | `scriptSplit` | 接 A 的 text，按编号拆成单条 |
| 视觉分支 | 故事板分镜生图器 C | `imageToImage` | 接 B 的 Text List + imageInput（广播） |

### 连线图

```text
imageInput ──────────────┐
       │                 │
       │                 ↓
textInput ───→ textGenerator A（多图描述集生成）
                      │
                      ↓
                scriptSplit B
                      │
                      ↓ [Text List]
               imageToImage C ←── imageInput（广播：1图×N描述→N张图）
                      │
                      ↓ [Image List]
                    输出
```

### 连线规则校验

- textInput text → A text ✅
- imageInput image → A image ✅
- A text → B text（scriptSplit.text 上限 1）✅
- B text(list) → C text ✅
- imageInput image → C image ✅（广播）
- DAG 无环 ✅

### 关键约束

- C 的 Image 输入**必须**来自 `imageInput`（不可用 Generated Image）
- A 的输出必须用稳定编号（Image 01、Image 02…），每块自洽

---

## Step 3：模型选择

| 节点 | 推荐模型 | 理由 |
|------|----------|------|
| A textGenerator | `google/gemini-3-pro-preview` | 需要图片理解能力（产品图分析） |
| B scriptSplit | `openai/gpt-5.2`（默认） | — |
| C imageToImage | `gemini-3-pro-image-preview`（Banana Pro）或 `fal-ai/gpt-image-1.5`（GPT Image 1.5） | Banana Pro 偏写实，GPT Image 偏可控/结构化。商品图常用这两个 |

---

## Step 4：提示词

### 缺口审计

| 节点 | 上游已有什么 | 缺什么 | 是否需要写 prompt |
|------|-------------|--------|------------------|
| A | 用户 textInput + imageInput | 角色设定 + 多图编号输出格式 | ✅ |
| B | A 的编号多图描述集 | — | ❌ 自动拆分 |
| C | B 的单条描述 + imageInput | 视觉控制（如需补构图/光影/风格）| 视情况轻量补充 |

### 节点 A 的 inputText（参照 text_prompt_best_practices 的 Master 指令）

```text
你是专业亚马逊 listing 视觉设计师。任务是生成编号多图描述集供下游生图。
每图标为 Image 01、Image 02…。每块自洽且仅描述一个图像目标。
区分不同图片职能：主图（白底产品）、卖点展示、使用场景、细节特写、对比图、包装/配件图等。
保持产品一致性。不解释不赘述，直接输出。每块不超过 1500 字。
```

### 节点 C 的 inputText（可选补充）

通常上游 A 的描述已足够完整，C 不需要额外 prompt。
如果需要统一视觉风格控制，可在 C 的 inputText 中补充：

```text
product studio lighting, clean white background, photorealistic, high detail
```

---

## Anti-patterns（本场景高频错误）

- ❌ C 使用 Generated Image 而非 imageInput → 退化为单图输出
- ❌ A 的输出不用编号 → scriptSplit 无法正确拆分
- ❌ A 把多张图的描述合并在一个块里 → 违反"一块一目标"
- ❌ 试图用一个 imageMaker 节点一次生成多张图 → 必须走 scriptSplit 拆分路径
- ❌ 不同图片之间产品描述不一致 → 每块应自洽，但产品信息应统一
