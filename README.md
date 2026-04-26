# bilibili-summary

Universal AI agent skill for transcribing Bilibili videos into Markdown reports.

**[🏠 GitHub](https://github.com/jianhu-chen/bilibili-summary)** | **[🇨🇳 中文文档](README.zh-CN.md)**

## ✨ Features

- [x] Fetches CC subtitles when available (faster, free)
- [x] Falls back to ASR transcription via OpenAI-compatible Whisper API (parallel processing for long videos)
- [x] AI-powered summary with Highlights + Detailed transcript sections
- [x] Auto audio segmentation for videos > 50 minutes
- [x] Auto dependency installation (yt-dlp, ffmpeg)
- [x] Customizable report language — defaults to the video's language; override by appending to the command or telling the agent in natural language (see [Language](#language))

## 📦 Installation

```bash
npx skills add jianhu-chen/bilibili-summary
```

Install to a specific agent:

```bash
# Claude Code
npx skills add jianhu-chen/bilibili-summary -a claude-code

# Cursor
npx skills add jianhu-chen/bilibili-summary -a cursor

# Codex
npx skills add jianhu-chen/bilibili-summary -a codex
```

Install globally (available in all projects):

```bash
npx skills add jianhu-chen/bilibili-summary -g
```

## 🚀 Usage

```
/bilibili-summary <bilibili-url>
```

Supported URL formats:
- `https://www.bilibili.com/video/BV1xx...`
- `https://www.bilibili.com/video/av12345`
- `https://b23.tv/xxxxx` (short links)
- Append `?p=N` for multi-part videos

## ⚙️ Configuration

| Environment Variable | Required | Default | Description |
|---------------------|----------|---------|-------------|
| `BILIBILI_COOKIES` | ✅ Yes | - | Bilibili cookie content in Netscape format. Required to access Bilibili videos |
| `ASR_API_KEY` | ✅ Yes | - | API key for Whisper-compatible ASR service |
| `ASR_MODEL` | No | `whisper-1` | ASR model name |
| `ASR_BASE_URL` | No | `https://api.openai.com/v1` | Custom API base URL |

### 🔑 How to Get Bilibili Cookies

Bilibili has anti-scraping measures. `BILIBILI_COOKIES` is **required** — without it, the tool will exit with an error. Setting it requires a **logged-in Bilibili account**. Cookies are valid for ~30 days.

**Method 1: yt-dlp export (Recommended)**

```bash
yt-dlp --cookies-from-browser chrome --cookies cookies.txt "https://www.bilibili.com/video/BV1MN4y177PB" --skip-download
export BILIBILI_COOKIES="$(cat cookies.txt)"
```

**Method 2: Browser extension**

1. Install [Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)
2. Log into bilibili.com, click the extension to export
3. `export BILIBILI_COOKIES="$(cat /path/to/cookies.txt)"`

## 🌐 Language

By default, reports are written in the same language as the video. Technical terms are preserved in their original language.

To specify a different language for the report, include your language preference in your request, e.g.:

```
/bilibili-summary <bilibili-url> Please write the report in English
```

## 📋 Dependencies

Missing dependencies will be auto-installed by the agent during the first run.

| Dependency | macOS | Linux (Debian/Ubuntu) |
|-----------|-------|----------------------|
| [yt-dlp](https://github.com/yt-dlp/yt-dlp) | `brew install yt-dlp` or `pip3 install yt-dlp` | `pip3 install yt-dlp` |
| [ffmpeg](https://ffmpeg.org/) | `brew install ffmpeg` | `sudo apt install ffmpeg` |
| Python 3 | Pre-installed or `brew install python3` | `sudo apt install python3` |
| curl | Pre-installed or `brew install curl` | `sudo apt install curl` |

## ⚡ How It Works

1. 🎬 **Metadata** — Fetches video info via yt-dlp
2. 💬 **Subtitles** — Tries to extract CC subtitles (skips ASR if found)
3. 📥 **Download** — Falls back to audio download via yt-dlp if no subtitles
4. ✂️ **Segment** — Splits audio into chunks if > 50 min, compresses to 32k mono
5. 🎙️ **Transcribe** — Calls Whisper ASR API in parallel (max 4 concurrent)
6. 🤖 **Summarize** — AI generates Highlights + detailed written transcript
7. 📄 **Output** — Saves a `.md` report in the current directory

## Related Projects

- [youtube-summary](https://github.com/jianhu-chen/youtube-summary) — Transcribe YouTube videos into Markdown reports

## 📄 License

MIT
