---
name: tiger-tv
description: "当任务涉及影视搜索、按来源和 vod_id 获取播放或下载链接、批量下载剧集，或生成 Quantumult X 直连规则时，优先使用此技能。"
triggers:
  - 影视搜索
  - 剧集下载
  - 播放/下载链接
  - Quantumult X
  - 直连规则
  - 代理规则
  - 小老虎爱看剧
---

# tiger-tv：小老虎爱看剧.skill

用于搜索影视剧集、获取播放/下载链接、批量下载剧集，以及生成 Quantumult X 直连规则。

## 安装

首次使用，或 `tiger-tv.py` 命令不存在时：

```bash
curl -fsSL https://raw.githubusercontent.com/qvshuo/TigerTV/main/install.sh | bash
```

搜索、获取链接和生成规则功能依赖 `tiger-tv.py`；下载功能还额外依赖 `yt-dlp` 和 `ffmpeg`。

## 使用

### 1. 搜索

```bash
tiger-tv.py search <keyword>
```

进度信息输出到 stderr，结果输出到 stdout。示例输出：

```
source: 🎬金鹰点播:
  vod_id: 104571
  vod_name: 逐玉
  vod_time: 2026-03-22 23:35:02
  vod_remarks: 第40集已完结
```

### 2. 获取链接

命令示例：

```bash
tiger-tv.py fetch --source "🎬金鹰点播" --vod_id 104571
```

--source 需完整匹配来源名称，--vod_id 为整数，二者均可从 `search` 的输出中获取。

### 3. 下载

**默认行为**：根目录为 `~/Downloads/小老虎爱看剧`，目录结构为 `剧名/Season N/SxxExx.mp4`。未指定季号时使用 `Season 1`；未指定来源时使用第一个可用资源；未指定集数时默认下载全部。

**链接分组**：`fetch` 的输出可能包含多组链接（例如第一组为 `mp4` 链接，第二组为 `.m3u8` 直链）。默认使用第一组进行下载；若第一组失败，则依次尝试后续分组。

**必须参数**：每次调用 `yt-dlp` 时都必须携带以下参数，以确保支持断点重试并提升下载稳定性：

```
--retries 20 --retry-sleep 5 \
--concurrent-fragments 16 --fragment-retries 20 \
--socket-timeout 30 --ignore-errors
```

**批量下载**：优先使用单次 `yt-dlp` 批量下载多集，以充分利用带宽；单次调用中，每集对应一组 `-o <输出路径> <URL>`。为兼顾效率与稳定性，单批最多下载 10 集；若需下载更多集数，则按每批最多 10 集拆分并依次执行。例如下载 40 集时，应分 4 次进行，每次下载 10 集，而不是逐集单独下载。

```bash
mkdir -p ~/Downloads/小老虎爱看剧/逐玉/Season\ 1
yt-dlp \
  --retries 20 --retry-sleep 5 \
  --concurrent-fragments 16 --fragment-retries 20 \
  --socket-timeout 30 --ignore-errors \
  -o "~/Downloads/小老虎爱看剧/逐玉/Season 1/S01E01.mp4" "https://example.com/ep01.m3u8" \
  -o "~/Downloads/小老虎爱看剧/逐玉/Season 1/S01E02.mp4" "https://example.com/ep02.m3u8" \
  -o "~/Downloads/小老虎爱看剧/逐玉/Season 1/S01E03.mp4" "https://example.com/ep03.m3u8"
# …每集追加一对 -o <路径> <URL>
```

### 4. 生成 Quantumult X 规则

```bash
tiger-tv.py quanx <keyword>
```

输出分为三段：资源站 API 域名、播放/下载域名，以及 m3u8 域名。

## 典型流程

```text
1. 搜索 -> 获取链接 -> 使用 yt-dlp 下载
tiger-tv.py search <keyword> -> tiger-tv.py fetch --source "..." --vod_id ... -> yt-dlp ...
2. 生成 Quantumult X 直连规则
tiger-tv.py quanx <keyword>
```
