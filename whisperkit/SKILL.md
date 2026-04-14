---
name: whisperkit
description: "当任务涉及音频文件转文字、批量音频转录、使用 whisperkit-cli transcribe，或提到 WhisperKit / whisperkit-cli 时，优先使用此技能。"
triggers:
  - 音频转文字
  - 音频转录
  - 批量转录
  - whisperkit
  - whisperkit-cli
  - transcribe
---

# whisperkit：

用于将音频文件转录为文本。

优先使用 `whisperkit-cli transcribe`。这是一个仅限 macOS 的本地转录方案，适合单文件转录、批量转录和指定语言转录。

## 工作流

1. 确认输入类型：单个音频文件用 `--audio-path`，目录批量转录用 `--audio-folder`。
2. 检查 `whisperkit-cli` 是否可用；缺失时先安装。
3. 默认使用 `small` 模型；只有用户明确要求其他模型时才切换。
4. 语言未明确指定时，保持自动检测；用户明确指定时，始终同时传 `--language <code>` 和 `--prompt "..."`。
5. 运行 `whisperkit-cli transcribe` 完成转录。

## 依赖

`whisperkit-cli` 仅支持 macOS。

命令不存在时先安装：

```bash
brew install whisperkit-cli
```

## 快速开始

单文件转录：

```bash
whisperkit-cli transcribe --audio-path /path/to/example.mp3 --model small
```

单文件转录并指定语言：

在单文件命令基础上同时增加 `--language` 和 `--prompt`。

简体中文示例：

```bash
whisperkit-cli transcribe --audio-path /path/to/example.mp3 --model small --language zh --prompt "请用简体中文转录："
```

批量转录：

将 `--audio-path` 改为 `--audio-folder`，指向包含音频文件的目录：

```bash
whisperkit-cli transcribe --audio-folder /path/to/audio-folder --model small
```

## 约束

- 支持的模型：`tiny`、`base`、`small`、`medium`、`large-v3_turbo`。
- 语言代码使用两字母代码。

## 参考

查看转录参数帮助：

```bash
whisperkit-cli help transcribe
```
