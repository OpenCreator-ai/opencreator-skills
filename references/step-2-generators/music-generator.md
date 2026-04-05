---
generator: 音乐生音频器
node_type: musicGenerator
input_modes: [text]
output_mode: audio
pattern: single
keywords: [音乐, 配乐, BGM, music, background]
---

# 音乐生音频器（Music Generator）

## 定义

将风格/情绪/场景的文本描述转化为可用的音乐内容。用于生成背景音乐，不承担主体表达信息。

## 输入 / 输出

- **输入**：Text（音乐描述，必选）
- **输出**：Audio（音乐音频）
- **可选参数**：Instrumental（是否纯音乐，推荐口播场景开启）

## 适用场景

- 视频背景音乐（广告 / UGC / 短视频配乐）
- 情绪氛围构建（紧张 / 轻松 / 治愈 / 科技感）
- 纯音乐（Instrumental）用于配音/口播场景

## 编排规则

```text
需要配乐 → 使用该模块
需要语音 → 使用 TTS / Voice Cloning，不用本模块
```

## 文本输入要点

描述应包含：风格（lofi / cinematic / upbeat）、情绪（happy / emotional）、使用场景（ad / background）。

## 关键原则

- Text 决定音乐风格，Instrumental 控制是否有人声
- 音乐长度可长于视频，后续剪辑中裁剪对齐
- 口播视频建议开启 Instrumental，避免影响语音清晰度

## 常见错误

- ❌ 期待精准控制歌词 → 该模块更适合纯音乐而非歌词驱动
- ❌ 未开启 Instrumental 用于口播视频 → 可能干扰语音
