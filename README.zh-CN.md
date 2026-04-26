# bilibili-summary

通用 AI agent skill，将哔哩哔哩视频转录为 Markdown 报告。

**[GitHub](https://github.com/jianhu-chen/bilibili-summary)** | **[English](README.md)**

## 功能

- 优先提取 CC 字幕（更快、免费）
- 无字幕时回退到 ASR 语音转文字（OpenAI 兼容 Whisper API，长视频自动并行处理）
- AI 生成包含要点摘要和视频详情的 Markdown 报告
- 超过 50 分钟的视频自动分段处理
- 自动安装缺失依赖（yt-dlp、ffmpeg）
- 可自定义报告语言 — 默认使用视频原语言，追加到命令后或直接用自然语言告诉 Agent 即可覆盖（详见[语言](#语言)）

## 安装

```bash
npx skills add jianhu-chen/bilibili-summary
```

安装到指定 agent：

```bash
# Claude Code
npx skills add jianhu-chen/bilibili-summary -a claude-code

# Cursor
npx skills add jianhu-chen/bilibili-summary -a cursor

# Codex
npx skills add jianhu-chen/bilibili-summary -a codex
```

全局安装（所有项目可用）：

```bash
npx skills add jianhu-chen/bilibili-summary -g
```

## 使用

```
/bilibili-summary <bilibili-url>
```

支持的 URL 格式：
- `https://www.bilibili.com/video/BV1xx...`
- `https://www.bilibili.com/video/av12345`
- `https://b23.tv/xxxxx`（短链接）
- 多 P 视频在 URL 后加 `?p=N`

## 配置

| 环境变量 | 必填 | 默认值 | 说明 |
|---------|------|-------|------|
| `BILIBILI_COOKIES` | **是** | - | Bilibili cookie 内容（Netscape 格式），访问 Bilibili 视频必须设置 |
| `ASR_API_KEY` | **是** | - | Whisper 兼容 ASR 服务的 API 密钥 |
| `ASR_MODEL` | 否 | `whisper-1` | ASR 模型名称 |
| `ASR_BASE_URL` | 否 | `https://api.openai.com/v1` | 自定义 API 基础 URL |

### 如何获取 Bilibili Cookie

Bilibili 有反爬机制，`BILIBILI_COOKIES` 为**必填项**，未设置时工具会直接报错退出。设置需要**已登录的 B站账号**，Cookie 有效期约 30 天。

**方式一：yt-dlp 导出（推荐）**

```bash
yt-dlp --cookies-from-browser chrome --cookies cookies.txt "https://www.bilibili.com/video/BV1MN4y177PB" --skip-download
export BILIBILI_COOKIES="$(cat cookies.txt)"
```

**方式二：浏览器扩展**

1. 安装 [Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)
2. 登录 bilibili.com，点击扩展导出
3. `export BILIBILI_COOKIES="$(cat /path/to/cookies.txt)"`

## 语言

默认情况下，报告使用与视频相同的语言撰写，技术术语保留原文。

如需指定其他语言，请在请求中说明，例如：

```
/bilibili-summary <bilibili-url> 请用英文撰写报告
```

## 依赖

缺失的依赖会在首次运行时由 Agent 自动安装。

| 依赖 | macOS | Linux (Debian/Ubuntu) |
|------|-------|----------------------|
| [yt-dlp](https://github.com/yt-dlp/yt-dlp) | `brew install yt-dlp` 或 `pip3 install yt-dlp` | `pip3 install yt-dlp` |
| [ffmpeg](https://ffmpeg.org/) | `brew install ffmpeg` | `sudo apt install ffmpeg` |
| Python 3 | 系统自带 或 `brew install python3` | `sudo apt install python3` |
| curl | 系统自带 或 `brew install curl` | `sudo apt install curl` |

## 工作原理

1. **元数据** — 通过 yt-dlp 获取视频信息
2. **字幕** — 尝试提取 CC 字幕（找到则跳过 ASR）
3. **下载** — 无字幕时通过 yt-dlp 获取音频
4. **分段** — 超过 50 分钟自动切分，所有分片统一压缩至 32k mono
5. **转录** — 并行调用 Whisper ASR API（最大并发 4）
6. **总结** — AI 生成要点摘要 + 详细文字版
7. **输出** — 在当前目录保存 `.md` 报告

## 许可证

MIT
