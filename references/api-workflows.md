---
name: opencreator-prod
description: >
  OpenCreator 正式环境 Workflow API 完整调用指南。
  覆盖：搜索模板 → 复制模板 → 查参数 → 收集用户入参 → 运行 → 轮询 → 返回结果。
  开箱即用，仅正式环境，无需关心测试环境。
---

# OpenCreator Workflow API — 正式环境

> 只管正式环境。读完这份文档，你就能完整跑通从搜模板到出结果的全链路。

---

## 0. 配置（抄这里就够了）

```
Base URL : https://api-prod.opencreator.io
API Key  : <你自己的 API Key，格式为 sk_xxx>
认证方式  : 所有请求 Header 加 X-API-Key: <你的 API Key>
```

> 📌 **API Key 获取方式**：登录 OpenCreator 平台 → Settings → API Keys → 创建并复制你的 Key。
> API Key 因账号而异，不要使用他人的 Key，也不要把自己的 Key 分享给他人。

---

## 1. 完整流程一览

```
① 搜索模板          GET  /api/developer/v1/templates?keyword=xxx
② 用户选择模板
③ 复制模板拿 flow_id POST /api/developer/v1/workflows/from-template
④ 查真实入参        GET  /api/developer/v1/workflows/{flow_id}/parameters
⑤ 向用户逐项确认入参（禁用默认值）
⑥ 运行 workflow     POST /api/developer/v1/workflows/{flow_id}/runs
⑦ 轮询状态          GET  /api/developer/v1/workflow-runs/{task_id}
⑧ 取结果并发给用户   GET  /api/developer/v1/workflow-runs/{task_id}/results

---

## 全部接口一览表

| 分类 | 接口 | 方法 | 说明 |
|---|---|---|---|
| 模板 | `/api/developer/v1/templates` | GET | 搜索模板（无分页） |
| 模板 | `/api/developer/v1/workflows/from-template` | POST | 复制模板生成 flow_id |
| 工作流 | `/api/developer/v1/workflows` | POST | 创建空工作流 |
| 工作流 | `/api/developer/v1/workflows/{flow_id}` | GET | 查询工作流详情（nodes/edges） |
| 工作流 | `/api/developer/v1/workflows/{flow_id}` | PATCH | 修改节点/边/名称 |
| 运行 | `/api/developer/v1/workflows/{flow_id}/parameters` | GET | Step 1：查运行参数 |
| 运行 | `/api/developer/v1/workflows/{flow_id}/runs` | POST | Step 2：运行 |
| 运行 | `/api/developer/v1/workflow-runs/{task_id}` | GET | Step 3：轮询状态 |
| 运行 | `/api/developer/v1/workflow-runs/{task_id}/results` | GET | Step 4：取结果 |
| 运行 | `/api/developer/v1/workflow-runs/{task_id}/cancel` | POST | 取消（当前节点继续扣费，后续停止） |
| 其他 | `/health` | GET | 健康检查 |
```

---

## 2. 搜索模板

### 调用

```bash
curl -s "https://api-prod.opencreator.io/api/developer/v1/templates?keyword=UGC" \
  -H "X-API-Key: $API_KEY"
```

### 关键字段

| 字段 | 说明 |
|---|---|
| `records[].template_id` | 模板 ID，后续复制用 |
| `records[].name.zh` | 模板中文名 |
| `records[].desc.zh` | 功能描述 |
| `records[].models` | 使用的 AI 模型 |
| `records[].covers[0]` | 效果图 URL（可选，发给用户预览） |

### 注意事项

- **无分页**：不传 `page_number`/`page_size`，一次返回全部匹配结果
- **无 `workflow_io`**：正式环境不返回 inputs/outputs 类型预览，Step ④ 查参数才知道
- **中文关键词搜不到时**：自动翻译成英文重搜（模板库以英文索引为主），仍无结果再告知用户

### 关键词提取规则

用户说需求时，背地里提取 1-3 个关键词传给接口，不要让用户感知搜索过程：

1. **优先取原始实词**：从用户原话里直接抠名词/动词
2. **语言跟着用户走**：用户说中文 → 传中文；说英文 → 传英文
3. **最多 3 个关键词**，逐个搜索取并集去重

| 用户说 | 提取关键词 |
|---|---|
| "帮我做珠宝亚马逊上架图" | `珠宝`、`亚马逊` |
| "I want viral UGC video" | `viral`、`UGC` |
| "帮我找 AI 生视频的" | `视频` |

### 结果排序与筛选（必做）

API 返回全量匹配结果，你必须在展示前做 client-side 排序，只向用户展示最相关的 **前 5 个**。

排序评分规则（按权重从高到低叠加）：

| 信号 | 判断方式 | 权重 |
|---|---|---|
| **输出类型匹配** | 用户要图就优先 output 含 image 的，要视频就优先 video | 最高 |
| **场景关键词命中** | 模板 `name` 或 `desc` 是否包含用户需求中的核心名词（如"UGC""电商""lipsync"） | 高 |
| **输入类型匹配** | 用户提供了图片则优先需要 image input 的模板，提供了视频则优先 video input | 中 |
| **模型质量** | 使用较新或高质量模型的排前面（如 Sora 2 > 旧模型） | 低 |

操作步骤：

1. 将多个关键词的搜索结果合并，按 `template_id` 去重
2. 对每个模板按上述规则打分排序
3. 取前 5 个展示给用户；如果总数不足 5 个就全部展示
4. 如果排序后没有高度匹配的模板，告知用户当前模板库无精确匹配，建议换关键词或进入 Build Mode

### 向用户展示格式

展示排序后的前 5 个模板，带序号方便用户选择：

```
1. **模板名称**
   功能描述（一句话）
   模型：xxx、xxx

2. **模板名称**
   ...
```

如果有效果图（`covers[0]`），优先附上让用户预览。

---

## 3. 复制模板拿 flow_id

> ⚠️ **必须复制**，不能直接用搜索结果里的 `origin_flow_id` 跑，公共模板只读。

```bash
curl -s -X POST "https://api-prod.opencreator.io/api/developer/v1/workflows/from-template" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_id": "template_xxx"}'
```

返回：`{ "flow_id": "xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" }`

> ⚠️ 如果返回 `edges contain invalid source/target` 错误 → 换另一个模板重试。

---

## 4. 查真实入参

> ⚠️ **每次必做，不可跳过。** 节点 ID 随时可能变化，不能硬编码。

```bash
curl -s "https://api-prod.opencreator.io/api/developer/v1/workflows/{flow_id}/parameters" \
  -H "X-API-Key: $API_KEY"
```

### 返回示例

```json
{
  "flow_id": "xxx",
  "inputs": [
    {
      "node_id": "textInput-abc123",
      "node_title": "产品描述",
      "input_type": "text",
      "required": false,
      "default_value": "..."
    },
    {
      "node_id": "imageInput-def456",
      "node_title": "Product Image",
      "input_type": "image",
      "required": false
    }
  ],
  "outputs": [
    {
      "node_id": "imageToImage-xxx",
      "output_type": "image",
      "is_default": true
    }
  ]
}
```

---

## 5. 向用户确认入参（必做，禁用默认值）

**🚨 核心原则：所有 input 节点，不论 `required=true` 还是 `false`，都必须向用户确认具体内容。禁止直接使用 `default_value` 跑任务。用户的需求是唯一依据。**

拿到 `inputs` 列表后，根据 `input_type` 逐项问用户：

| input_type | 需要用户提供 |
|---|---|
| `text` | 文本内容（根据 node_title 语义告知用户要填什么） |
| `image` | 图片文件或图片直链 URL |
| `video` | 视频文件或视频直链 URL |
| `audio` | 音频文件或音频直链 URL |

### 🚨 面客名称规则

**禁止把技术字段名暴露给用户。** `node_id`、`node_type`、`field_key`、`inputText`、`imageBase64` 等字段只在内部用，跟用户沟通时统一用业务语言。

| 原始 node_title | ❌ 错误说法 | ✅ 正确说法 |
|---|---|---|
| `Text Input` | "请填写 textInput 字段" | "请描述你的产品" |
| `Image Input` | "请提供 imageInput 图片" | "请上传你的商品图" |
| `Reference Video` | "Reference Video 节点需要视频" | "请发给我你想复刻风格的参考视频" |
| `node_id: abc-123` | 任何场合都不出现 | 永远不提 |

**示例问法（3个输入时）：**
> 这个工作流需要你提供 3 样东西：
> 1. **你的产品图** — 发图片文件或图片链接
> 2. **参考视频** — 你想复刻哪条爆款视频的节奏？发视频或链接
> 3. **产品信息** — 品名、核心卖点、目标市场、投放平台

**如果用户在初始需求中已提供了部分信息，推断后只补问缺失的，不重复问已有信息。**

---

## 6. 运行 workflow

### 调用

```bash
curl -s -X POST "https://api-prod.opencreator.io/api/developer/v1/workflows/{flow_id}/runs" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {
      "textInput-abc123": "你的文本内容",
      "imageInput-def456": "https://example.com/image.png",
      "videoInput-ghi789": "https://example.com/video.mp4"
    },
    "start_ids": [],
    "end_ids": []
  }'
```

### ✅ inputs 格式（踩坑重灾区）

**inputs 的 key = `node_id`（从 Step ④ 拿），value = 实际内容（字符串）**

```json
{
  "inputs": {
    "textInput-abc123": "你的文本",
    "imageInput-def456": "https://example.com/image.png"
  }
}
```

### ❌ 会卡 queued 的错误格式

```json
{
  "inputs": {
    "textInput-abc123": {"inputText": "你的文本"}
  }
}
```

> 用 `field_key` 多包一层 → 任务永远卡 `queued`，不报错、不运行。

返回：`{ "task_id": "TASK_XXXXXX", "status": "queued" }`

---

## 7. 轮询状态（强制闭环，不可中断）

### 🚨 核心义务：运行后必须轮询到终态，然后立刻取结果发给用户

**提交 run 之后，你的唯一下一步就是进入轮询循环。不允许停下来等用户催你。**

轮询闭环协议：

1. 调用 run API 拿到 `task_id` 后，立刻告诉用户：「已提交，任务 ID 为 {task_id}，正在为你跟踪进度…」
2. 进入轮询循环，每次轮询后向用户报告当前状态
3. 直到状态变为 `success` / `failed` / `cancelled` 才停止
4. `success` → **立刻**调 results API，**立刻**把媒体发给用户
5. `failed` → 读 `error_message` 和 `node_statuses`，告知用户失败原因并建议下一步
6. **绝对不允许**在 `queued` 或 `running` 状态时停止轮询、等用户来问

```bash
curl -s "https://api-prod.opencreator.io/api/developer/v1/workflow-runs/{task_id}" \
  -H "X-API-Key: $API_KEY"
```

### 状态说明

| 状态 | 含义 | 下一步 |
|---|---|---|
| `queued` | 排队中 | 继续轮询，告知用户「排队中，请稍候…」 |
| `running` | 运行中 | 继续轮询，告知用户「生成中，请稍候…」 |
| `success` | 成功 | **立刻** → Step ⑧ 取结果并发给用户 |
| `failed` | 失败 | 查 `error_message` + `node_statuses`，告知用户 |
| `cancelled` | 已取消 | 告知用户任务已取消 |

### 轮询频率

- 生文 / 生图 workflow：**每 10 秒**查一次
- 生视频 / 多节点链路：**每 30 秒**查一次

### 每次轮询时的自我提醒格式

为了防止在长上下文中丢失任务追踪，每次轮询时在内部记录：

```
[轮询] task_id={task_id} | flow_id={flow_id} | 当前状态={status} | 已轮询={N}次
```

### 轮询超时处理

| 场景 | 判断条件 | 动作 |
|---|---|---|
| queued 卡死 | 连续轮询超过 5 分钟仍为 `queued` | 检查 `input_overrides` 是否有双重嵌套；如有，修正 inputs 重跑 |
| running 超长 | 生图超过 5 分钟 / 生视频超过 15 分钟仍为 `running` | 告知用户当前仍在运行，继续等待；超过 30 分钟建议取消重试 |
| 反复失败 | 连续 2 次 run 都 `failed` | 停止重试，展示错误信息，建议用户换模板或调整输入 |

### queued 卡死诊断

任务卡 `queued` 超过 5 分钟 → 大概率是 inputs 格式错了（双重嵌套），用以下命令确认：

```bash
curl -s "https://api-prod.opencreator.io/api/developer/v1/workflow-runs/{task_id}" \
  -H "X-API-Key: $API_KEY" | python3 -c "
import json,sys
d=json.load(sys.stdin)
print('status:', d['status'])
print('input_overrides:', json.dumps(d.get('input_overrides',{}), indent=2, ensure_ascii=False))
"
```

如果看到 `{"inputText": {"inputText": "..."}}` → inputs 格式错了，改成扁平格式重跑。

---

## 8. 取结果 + 发给用户

### 调用

```bash
curl -s "https://api-prod.opencreator.io/api/developer/v1/workflow-runs/{task_id}/results" \
  -H "X-API-Key: $API_KEY"
```

### 返回示例（完整字段）

```json
{
  "task_id": "TASK_xxx",
  "flow_id": "xxx",
  "run_id": "xxx",
  "status": "success",
  "outputs": [
    {
      "node_id": "imageToImage-xxx",
      "node_type": "imageToImage",
      "node_title": "UGC Video",
      "output_type": "video",
      "results": [
        {
          "result_id": "xxx",
          "type": "video",
          "content": "https://resource.opencreator.io/videos/xxx.mp4",
          "model": "Sora 2",
          "status": "success",
          "credits_used": 2007,
          "created_at": "2026-04-02 18:50:00"
        }
      ]
    }
  ],
  "total_credits_used": 2007
}
```

关键字段：
- `outputs[].results[].content` = 结果内容（图片/视频/音频是 URL，文本是文本）
- `outputs[].results[].model` = 实际使用的模型名
- `total_credits_used` = 本次任务总积分消耗

### 🚨 结果必须直接发给用户（不能只发链接文字）

| output_type | 做法 |
|---|---|
| `image` | 用 `message` tool 发图片（`media` 参数传图片 URL），用户直接看到图 |
| `video` | 用 `message` tool 发视频（`media` 参数传视频 URL），用户直接播放 |
| `audio` | 用 `message` tool 发音频（`media` 参数传音频 URL） |
| `text` | 直接在消息正文输出文本内容 |

**同时附上可下载的直链**：`原文件下载：https://...`

❌ 错误：「生成成功，链接在这：https://...」
✅ 正确：用 message tool 发媒体文件 + 附下载链接

---

## 9. 媒体文件托管（用户发来的图片/视频需上传）

OpenCreator 输入只接受直链 URL，本地文件需先上传托管。

| 优先级 | 服务 | 适用 | 命令 |
|---|---|---|---|
| 1 | **tmpfiles.org** | 视频 + 图片 | `curl -F "file=@file.mp4" https://tmpfiles.org/api/v1/upload` |
| 2 | **catbox.moe** | 仅图片 | `curl -F "reqtype=fileupload" -F "fileToUpload=@file.png" https://catbox.moe/user/api.php` |
| 3 | 告知用户 | — | 全部失败时暂停，请用户提供直链 |

> ⚠️ **tmpfiles 必须把 URL 改成 `/dl/` 直链**：
> `http://tmpfiles.org/12345/file.mp4` → `http://tmpfiles.org/dl/12345/file.mp4`

> ⚠️ catbox.moe 不支持视频（实测失败），视频只用 tmpfiles。

---

## 10. 工作流管理接口

### 创建空工作流

```bash
curl -s -X POST "https://api-prod.opencreator.io/api/developer/v1/workflows" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "我的工作流名称"}'
```

返回：`{ "flow_id": "xxx", "flow_name": "xxx", "created_at": "..." }`

创建后再通过 PATCH 接口填入 nodes/edges（Node/Edge 结构参考 create-workflow skill）。

### 查询工作流详情

```bash
curl -s "https://api-prod.opencreator.io/api/developer/v1/workflows/{flow_id}" \
  -H "X-API-Key: $API_KEY"
```

返回工作流完整的 `nodes` 和 `edges` 数组，可用于读取现有结构后做修改。

### 修改工作流（PATCH）

```bash
curl -s -X PATCH "https://api-prod.opencreator.io/api/developer/v1/workflows/{flow_id}" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "新名称（可选，不传则不改）",
    "nodes": [...],
    "edges": [...]
  }'
```

> ⚠️ `nodes` 和 `edges` **要么都不传，要么必须一起传**

### 取消运行中的任务

```bash
curl -s -X POST "https://api-prod.opencreator.io/api/developer/v1/workflow-runs/{task_id}/cancel" \
  -H "X-API-Key: $API_KEY"
```

> ⚠️ 调用后：当前正在运行的节点会**继续运行并扣费**，后续节点才停止。

---

## 11. 完整 curl 调用模板（复制可用）

```bash
export API_KEY='sk_xxx'   # ← 替换成你自己的 API Key
export BASE='https://api-prod.opencreator.io'

# ① 搜索模板
curl -s "$BASE/api/developer/v1/templates?keyword=UGC" \
  -H "X-API-Key: $API_KEY" | python3 -m json.tool

# ② 复制模板
FLOW_ID=$(curl -s -X POST "$BASE/api/developer/v1/workflows/from-template" \
  -H "X-API-Key: $API_KEY" -H "Content-Type: application/json" \
  -d '{"template_id": "template_xxx"}' \
  | python3 -c "import json,sys; print(json.load(sys.stdin)['flow_id'])")
echo "flow_id: $FLOW_ID"

# ③ 查参数
curl -s "$BASE/api/developer/v1/workflows/$FLOW_ID/parameters" \
  -H "X-API-Key: $API_KEY" | python3 -m json.tool

# ④ 运行（根据查到的 node_id 填 inputs）
TASK=$(curl -s -X POST "$BASE/api/developer/v1/workflows/$FLOW_ID/runs" \
  -H "X-API-Key: $API_KEY" -H "Content-Type: application/json" \
  -d '{
    "inputs": {
      "node-id-1": "文本内容",
      "node-id-2": "https://image-url.jpg",
      "node-id-3": "https://video-url.mp4"
    },
    "start_ids": [], "end_ids": []
  }' | python3 -c "import json,sys; print(json.load(sys.stdin)['task_id'])")
echo "task_id: $TASK"

# ⑤ 轮询（视频 workflow 用 30s，生图用 10s）
for i in $(seq 1 20); do
  sleep 30
  STATUS=$(curl -s "$BASE/api/developer/v1/workflow-runs/$TASK" \
    -H "X-API-Key: $API_KEY" \
    | python3 -c "import json,sys; print(json.load(sys.stdin)['status'])")
  echo "[$(date -u +%H:%M:%S)] $STATUS"
  [ "$STATUS" = "success" ] || [ "$STATUS" = "failed" ] && break
done

# ⑥ 取结果
curl -s "$BASE/api/developer/v1/workflow-runs/$TASK/results" \
  -H "X-API-Key: $API_KEY" | python3 -c "
import json,sys
d=json.load(sys.stdin)
for o in d['outputs']:
    for r in o.get('results',[]):
        if r.get('status')=='success':
            print(o['output_type'], ':', r['content'])
"
```

---

## 禁止事项

- ❌ 不查参数直接运行（节点 ID 会变）
- ❌ inputs value 用 field_key 包一层（会卡 queued）
- ❌ 直接用 `origin_flow_id` 跑（公共模板只读，必须先复制）
- ❌ 把网页链接当媒体 URL 传入（必须是可直接访问的文件直链）
- ❌ status 不是 success 就调 results
- ❌ catbox 传视频
- ❌ tmpfiles URL 忘记改成 `/dl/` 前缀
- ❌ 把技术字段名（node_id、inputText 等）暴露给用户
