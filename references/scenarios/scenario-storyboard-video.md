# 场景贯穿：多镜头分镜视频（广播路径）

> 本文件展示一个完整场景从 Step 1 到 Step 4 的全链路。适用于：用户有一段文字创意 + 一张人物/场景参考图，想生成一条多镜头叙事短视频。

---

## Step 1：结构反推

### 最终产物

多镜头叙事视频（>15 秒，多段镜头组成）

### 反推过程

```text
多镜头视频
→ 需要：多段视频片段（逐镜头）
→ 多段视频从哪来？→ 需要分镜图列表 + 分段文本列表 → 逐镜头图→视频
→ 分镜图列表从哪来？→ 需要分镜生图（广播：1 张参考图 × N 段文本 → N 张图）
→ 分段文本列表从哪来？→ 需要 scriptSplit ← 结构化脚本
→ 结构化脚本从哪来？→ 需要生文器 ← 用户文字创意
→ 参考图从哪来？→ 用户上传
```

### 抽象结构

```text
输入层（文字创意 + 参考图）
↓
语义层（结构化分镜脚本）
↓
分镜结构层（scriptSplit → 文本列表）
↓
视觉分支（1 张参考图 × N 段文本 → N 张分镜图）[广播]
↓
视频生成（1 张参考图 × N 段文本 → N 段视频）[广播]
↓
输出（视频列表）
```

### 追问检查

- 用户提供了参考图吗？→ 没有则需先生成一张，或追问
- 大约想要几个镜头？→ 影响脚本结构
- 偏好速度还是质量？→ 影响模型选择

---

## Step 2：选择 Generator 并连线

| 抽象模块 | Generator | node.type | 说明 |
|----------|-----------|-----------|------|
| 语义层 | 参考文本生文器 A | `textGenerator` | 接 textInput，输出编号分镜脚本（Shot 01/02/03…） |
| 分镜结构层 | 故事板分文器 B | `scriptSplit` | 接 A 的 text，输出 Text List |
| 视觉分支 | 故事板分镜生图器 C | `imageToImage` | 接 B 的 Text List + imageInput（广播） |
| 视频生成 | 参考图分镜生视频器 D | `videoMaker` | 接 B 的 Text List + imageInput（广播） |

> **注意**：这里有两种路径可选——
> - **路径 1（先图后视频）**：B → C(分镜图) → D(分镜视频，编号对齐)
> - **路径 2（直接图→视频，跳过分镜图）**：B + imageInput → D(广播)
>
> 路径 1 画面可控性更强（先锁定每镜画面再做动效），路径 2 更快更省节点。下方以路径 1 为例。

### 连线图（路径 1：先图后视频）

```text
textInput ───→ textGenerator A（分镜脚本生成）
                      │
                      ↓
                scriptSplit B
                      │
                      ↓ [Text List]
               imageToImage C ←── imageInput（广播：1图×N文本→N图）
                      │
                      ↓ [Image List]
                videoMaker D ←── Text List from B（编号对齐：Image#i + Text#i → Video#i）
                      │
                      ↓ [Video List]
                    输出
```

### 连线规则校验

- A text → B text（scriptSplit.text 上限 1）✅
- B text(list) → C text ✅
- imageInput image → C image ✅（广播，因为 imageInput 不参与编号对齐）
- B text(list) → D text ✅
- C image(list) → D image（编号对齐，数量必须一致）✅
- DAG 无环 ✅

### 关键约束

- C 的 Image 输入**必须**来自 `imageInput`（不可用 Generated Image，否则退化为单图）
- D 的 Image List 来自 C 的生成结果（此时是列表态，走编号对齐）
- B text(list) 数量 = C image(list) 数量 = D video(list) 数量

---

## Step 3：模型选择

| 节点 | 推荐模型 | 理由 |
|------|----------|------|
| A textGenerator | `openai/gpt-4o-2024-11-20` | 纯文本分镜脚本，结构稳定 |
| B scriptSplit | `openai/gpt-5.2`（默认） | — |
| C imageToImage | `gemini-3-pro-image-preview`（Banana Pro）| 写实，保持人物一致性 |
| D videoMaker | `fal-ai/sora-2/image-to-video` 或 `fal-ai/bytedance/seedance/v1.5/pro/image-to-video` | 画质与运动真实感强 |

### 视频时长拆分

```text
总需求时长 30s ÷ 5 个镜头 = 每个镜头约 6s
→ 选 duration: 5s-6s（最稳区间）
```

---

## Step 4：提示词

### 缺口审计

| 节点 | 上游已有什么 | 缺什么 | 是否需要写 prompt |
|------|-------------|--------|------------------|
| A | 用户原始 textInput | 角色设定 + 分镜输出格式 | ✅ |
| B | A 的分镜脚本 | — | ❌ scriptSplit 自动拆分 |
| C | B 的单条文本 + imageInput | 视觉控制（构图/景别/光影/风格）| 视情况补充 |
| D | B 的单条文本 + C 的分镜图 | 动态控制（动作/运镜）| 视情况补充 |

### 节点 A 的 inputText（参照 text_prompt_best_practices）

```text
你是专业分镜师与商业片视觉策划。任务是生成编号分镜脚本。
每镜标为 Shot 01、Shot 02…。每镜仅描述一个画面时刻。
每块包含：主体动作、环境、机位、运镜、情绪。
每块自洽，不写「同上」。不解释不赘述，直接输出。每块不超过 1500 字。
```

### 节点 C 的 inputText（可选补充，参照 image_prompt_best_practices）

如果上游文本已经包含完整视觉描述（主体+环境+构图+光影+风格），则不需要额外 prompt。
如果缺视觉控制维度，只补缺失部分：

```text
eye-level angle, rule of thirds composition, soft natural lighting, cinematic photorealistic style
```

### 节点 D 的 inputText（可选补充，参照 video_prompt_best_practices）

如果上游已有动作/运镜描述，不需要额外 prompt。
如果缺动态信息，只补：

```text
slow push-in, slight hand gesture, expression shifts from neutral to confident
```

---

## Anti-patterns（本场景高频错误）

- ❌ C 使用 Generated Image 而非 imageInput → 退化为单图输出，不会广播
- ❌ A 的脚本不用编号（Shot 01/02）→ scriptSplit 无法正确拆分
- ❌ A 的脚本中多个镜头合并在一块 → 违反"一块一目标"原则
- ❌ D 的 Image List 与 Text List 数量不一致 → 编号对齐失败
- ❌ C 或 D 的 prompt 重复描述主体外貌（imageInput 已经提供了）→ 造成不稳定
