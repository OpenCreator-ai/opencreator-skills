# Text to Speech – 模型选型知识

## 概述

Text to Speech 用于将文本转换为语音音频。

它主要用于生成：
- 配音（voiceover）
- 旁白（narration）
- 对话音频
- 口播语音轨
- 广告语音
- 本地化内容语音

本模块支持多个语音提供方，不同模型各自有明确的优势：

- **Fish Audio** → 名人模仿 / parody / 创作者风格语音
- **ElevenLabs v2** → 专业、品牌安全、适合旁白与广告的语音
- **Minimax Speech 2.8 HD** → 更适合中文内容与更自然的中文表达

---

## 输入（Input）

| 模态 | 说明 | 限制 |
|---|---|---|
| **Text（必选）** | 需要被转换为语音的文本脚本 | 最佳实践：单段文本建议控制在可生成 **30秒以内** 的音频长度 |

> **注意：** 如果原始脚本较长，建议先拆成多个较短片段，再并行生成语音。

---

## 输出（Output）

| 模态 | 说明 | 用途 |
|---|---|---|
| **Audio** | 使用选定语音模型与音色生成的语音音频 | 配音、旁白、lip-sync 输入、广告、对话 |

---

## Confirmed model IDs

只使用下面这些已确认归属于 `textToSpeech` 的模型 ID。

| 展示名 | 精确 model id |
|---|---|
| Fish Audio | `fish-audio/speech-1.6` |
| ElevenLabs v2 | `fal-ai/elevenlabs/tts/multilingual-v2` |
| MiniMax Speech 2.8 HD | `fal-ai/minimax/speech-2.8-hd` |
| MiniMax Speech 2.8 Turbo | `fal-ai/minimax/speech-2.8-turbo` |

硬规则：

- `selectedModels` 只能填上表中的精确 id
- `voice_effect` 负责选音色，不要把音色名误写成 model id
- 没有映射到 `textToSpeech` 的 TTS 模型不要自行猜测接入

---

## 模型选型

### 1. Fish Audio

**适合场景：**
- 名人模仿 / parody 风格
- meme 内容
- 创作者风格 / 网感内容
- 娱乐化、夸张化、风格明显的语音输出

#### 产品中常见音色示例

- Elon Musk
- Donald Trump
- Taylor Swift
- Alle
- Paula
- Energetic Male
- Mr Beast
- ElevenLabs Adam
- Marcus Narrator
- E-Girl Soft
- Horror Narrator

> **说明：** Fish Audio 的音色范围较广，可能包含名人灵感、创作者灵感，以及风格化的互联网语音角色。

---

### 2. ElevenLabs v2

**适合场景：**
- 品牌旁白
- 精致广告
- 解说视频
- 专业、中性、稳定的语音生成

#### 产品中常见音色示例

- Rachel
- Aria
- Charlotte
- River
- Charlie
- Callum
- Brian
- Bill
- Lily
- Sarah
- Roger
- Laura

> **说明：** ElevenLabs 的音色通常比风格化或 parody 型语音更稳定、更中性，也更适合正式生产环境。

---

### 3. fal-ai/minimax/speech-2.8-hd

**适合场景：**
- 中文语音生成
- 中文本地化内容
- 双语工作流
- 需要更自然中文流畅度与发音准确度的旁白

#### 产品中常见音色示例

- Lively Girl
- Patient Man
- Young Knight
- Determined Man
- Lovely Girl
- Decent Boy

#### 额外控制参数

| 参数 | 说明 |
|---|---|
| `language_boost` | 提升特定语言表现，例如中文 |
| `voice_speed` | 控制语速 |
| `voice_volume` | 控制输出音量 |

---

## 参数

### 通用参数

| 参数 | 说明 |
|---|---|
| `model` | 选择语音生成提供方 |
| `voice_effect` | 在该提供方内部选择具体音色 / 风格 |

### Minimax 专属参数

| 参数 | 说明 |
|---|---|
| `language_boost` | 提升所选语言表现 |
| `voice_speed` | 调整语速 |
| `voice_volume` | 调整音量 |

---

## 最佳实践

### 1. 控制单段时长

最佳实践：
- 每段文本建议生成 **30秒以内** 的音频

原因：
- 生成更快
- 稳定性更好
- 更适合并行生产
- 更方便后续 lip-sync 与剪辑对齐

---

### 2. 先拆长脚本

推荐方式：

`Long script → Text Splitter → short text chunks → parallel TTS generation`

尤其适合：
- 长旁白
- 多场景广告脚本
- narration 管线
- 多段口播视频

---

### 3. 根据内容类型匹配模型

| 场景 | 推荐模型 |
|---|---|
| 名人模仿 / parody / meme / 创作者风格 | Fish Audio |
| 专业品牌旁白 / 广告 / 解说 | ElevenLabs v2 |
| 中文旁白 / 中文本地化内容 | Minimax Speech 2.8 HD |

---

### 4. 写“给人说”的文本，而不是“给人读”的文本

为了得到更好的输出：
- 使用更短的句子
- 加入自然停顿
- 避免过于密集的长段文本
- 尽量用口语化、可说出口的节奏来写

---

## 常见用途

- **UGC 广告配音**
- **短视频旁白**
- **创作者风格 AI 语音生成**
- **品牌解说语音**
- **中文本地化**
- **Lip-sync 音频生成**
- **多分镜批量语音生成**

---

## 限制

- 长文本块会降低稳定性并增加生成时间
- 多段独立生成的语音在一致性上可能会有轻微波动
- 风格化语音可能会放大语气或表达方式
- 最终时间轴通常仍需要后续编辑来实现精确同步
