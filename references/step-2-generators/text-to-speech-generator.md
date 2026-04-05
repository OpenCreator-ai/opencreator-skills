---
generator: 文本转语音生音频器
node_type: textToSpeech
input_modes: [text]
output_mode: audio
pattern: single
keywords: [TTS, 配音, 语音, 旁白, text to speech]
---

# 文本转语音生音频器（Text to Speech Generator）

## 定义

将文本内容转化为语音音频。是口播、配音、旁白等场景的基础模块，也是 lip-sync 的上游依赖。

## 输入 / 输出

- **输入**：Text（必选）
- **输出**：Audio（语音音频）

## 适用场景

- 口播音频生成（UGC / 广告）
- 旁白（解说类视频、storytelling）
- 多语言语音生成（本地化内容）

## 编排规则

```text
有文本 + 需要语音 → 使用该模块
已有音频 → 不使用
需要特定音色克隆 → 使用音色转语音生音频器 (voiceCloner)
```

## 关键原则

- 文本决定"说什么"，音频长度由文本长度决定
- 单段文本建议控制在可生成 **≤30 秒** 的音频长度，保证稳定性
- 长脚本建议先走 `scriptSplit` 拆分，再并行 TTS
- 常用于 lip-sync 上游：TTS → imageAudioToVideo 或 TTS → videoLipSync

## 常见错误

- ❌ 输入为空或不完整 → 无法生成有效语音
- ❌ 一次生成超长音频 → 稳定性下降，应先拆分
