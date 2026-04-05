---
generator: 音色转语音生音频器
node_type: voiceCloner
input_modes: [audio, text]
output_mode: audio
pattern: single
keywords: [音色克隆, 声音复刻, voice cloning]
---

# 音色转语音生音频器（Voice Cloning Generator）

## 定义

用指定音色参考朗读新文本，实现声音风格复刻。与普通 TTS 的区别：TTS 用预设音色，本模块用用户提供的音色参考。

## 输入 / 输出

- **输入**：Text（台词，必选）+ Audio（音色参考，必选）
- **输出**：Audio（克隆音色后的语音）

## 适用场景

- 克隆特定声音读新文本
- 统一品牌声音（广告 / narration）
- 角色语音复用（同一角色多段台词保持一致）

## 编排规则

```text
有音色参考 + 有文本 → 使用该模块
无音色参考 → 使用普通 textToSpeech
```

## 核心机制

```text
内容（Text）决定说什么 + 音色（Audio）决定用谁的声音
→ 输出长度由文本决定，与输入音频长度无关
```

## 关键原则

- Text 控制内容，Audio 控制音色，两者职责分离
- 单段文本建议 ≤30 秒，长脚本先 `scriptSplit` 再并行克隆
- 参考音频需清晰、少底噪

## 常见错误

- ❌ 只输入音频不输入文本 → 无法生成新内容
- ❌ 期待复制语义 → 该模块只复制声音，不复制内容
- ❌ 音色参考过短/质量差 → 会影响克隆效果
