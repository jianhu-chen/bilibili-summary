---
name: bilibili-summary
description: "Transcribe Bilibili videos into Markdown reports. Fetches CC subtitles when available, falls back to ASR via Whisper API."
argument-hint: "<bilibili-url> [language preference]"
allowed-tools: Bash(python3 *) Bash(yt-dlp *) Bash(ffmpeg *) Bash(brew *) Bash(pip3 *) Bash(curl *) Bash(mktemp *)
---

# Bilibili Video Transcription Report Generator

Convert Bilibili videos into Markdown text reports. Tries to fetch CC subtitles first; falls back to ASR (Whisper API) only when subtitles are unavailable.

## Input

Bilibili URL: `$1` (first space-separated token from user input). Supported formats:
- `https://www.bilibili.com/video/BV1xx...`
- `https://www.bilibili.com/video/av12345`
- `https://b23.tv/xxxxx` (short links)
- Append `?p=N` for multi-part videos

The full `$ARGUMENTS` may contain additional text beyond the URL — treat it as user instructions (e.g., language preference).

## Workflow

To avoid long-running timeouts, the process is split into multiple steps. Each step is independent.

### 1. Create a unique working directory

```bash
WORKDIR=$(mktemp -d /tmp/bili-summary-XXXXXX)
echo "Workdir: $WORKDIR"
```

Remember `$WORKDIR` — it is used in all subsequent steps.

### 2. Prepare: fetch metadata + try subtitles (or fallback to audio download + chunking)

```bash
python3 "./scripts/transcribe.py" prepare "$1" "$WORKDIR"
```

On success, stdout outputs a compact JSON. Key fields:

| Field | Description |
|-------|-------------|
| `subtitle_found` | `true` if CC subtitles were extracted, `false` if using ASR fallback |
| `chunk_count` | Number of audio chunks (0 when subtitles are used) |
| `workdir` | Working directory path |
| `title` | Video title |
| `uploader` | UP主 |
| `upload_date` | Upload date |
| `duration` | Video duration |
| `url` | Video URL |

Read and remember `subtitle_found` and `chunk_count`. Full metadata is saved in `$WORKDIR/metadata.json`.

**Handling missing dependencies** (non-zero exit code):

| Exit code | Meaning | Fix |
|-----------|---------|-----|
| 10 | yt-dlp not installed | Run `pip3 install yt-dlp` (or `brew install yt-dlp` on macOS), then retry |
| 11 | ffmpeg not installed | Run `sudo apt install ffmpeg` on Linux or `brew install ffmpeg` on macOS, then retry |
| 12 | ASR_API_KEY not set | Tell the user to set environment variable `ASR_API_KEY` |
| 13 | curl not installed | Install curl via system package manager, then retry |
| 14 | Bilibili anti-scraping block (HTTP 412) | Cookie may be expired, update `BILIBILI_COOKIES` with fresh cookie content, then retry |
| 15 | BILIBILI_COOKIES not set | Set environment variable `BILIBILI_COOKIES` with your cookie content (Netscape format) |

### 3. If `subtitle_found` is `true`

Skip to **Step 5** — the transcript is already prepared from CC subtitles.

### 4. If `subtitle_found` is `false`: Transcribe via ASR

```bash
python3 "./scripts/transcribe.py" transcribe-all "$WORKDIR"
```

Transcribe all chunks in parallel (max concurrency 4). The script prints progress like `[Progress] 2/3 chunks done`. If a chunk fails, retry individually with `transcribe "$WORKDIR" <index>`.

### 5. Collect: merge all transcription text

```bash
python3 "./scripts/transcribe.py" collect "$WORKDIR"
```

stdout outputs the full transcript text (from subtitles or ASR). Save it as transcript.

### 6. Generate Markdown report

Read `$WORKDIR/metadata.json` for full metadata (description, tags, etc. help understand the video content), then combine with the transcript to write the report and save it in the current working directory.

**Filename**: Based on the video content, use the LLM to generate a concise title (max 30 characters, summarizing the core theme of the video) as the filename `{summarized-title}.md` (strip illegal characters `\/:*?"<>|`). Do NOT use the original title directly.

**Report structure**:

```markdown
# {Bilibili original title}

> **Uploader**: {uploader} | **Date**: {upload_date} | **Duration**: {duration}
> **URL**: {url}

## Highlights

- {key point 1}
- {key point 2}
- ... (extract the most critical points, no more than 10, keep it concise)

## Video Details

{Organize as a polished written version following the original video's narrative order}

### {Topic 1}

{Detailed content...}

### {Topic 2}

{Detailed content...}
```

Where `{report_generated_at}` is the current date and time when the report is generated, formatted as `YYYY-MM-DD HH:MM`.

### 7. Writing guidelines

**Highlights section**:
- Extract the most critical points from the video, concise and refined
- One sentence per point, highlight key information
- Preserve data, conclusions, and core arguments

**Video Details section**:
- Preserve the original video's narrative structure and information density as much as possible
- Only apply the following treatments:
  - Remove subjective expressions, filler words, and pauses (e.g. "嗯", "那个", "就是说", "sort of", "you know", etc.)
  - Remove redundant phrasing
  - Convert spoken language into polished written form
- **Do NOT heavily compress the content** — preserve the full reasoning logic and information volume
- Split naturally by topic transitions in the video, using `###` subheadings
- Preserve data, quotes, arguments, and case studies from the video

**Ad handling**:
- Skip ads at the beginning or end of the video, sponsor reads, and promotional content
- Bilibili-specific ad patterns to skip:
  - "一键三连", "充电", "恰饭" content
  - "关注公众号", "扫码加群", "领资料"
  - "感谢赞助", "点击链接", "用我的优惠码"
- If ad segments are inserted in the middle of the video, skip them as well
- Do not mention any ad content in the report

**General rules**:
- **Language**: By default, write reports in the **same language as the video**. Preserve technical terms in their original language. If the full user input (`$ARGUMENTS`) contains text beyond the URL (e.g., "Please write the report in English" or "请用繁体中文撰写报告"), follow that language preference instead.
- If the video is short (< 5 minutes), the structure can be simplified

### 8. Clean up temporary files

After the report is written, clean up the working directory:

```bash
python3 "./scripts/transcribe.py" cleanup "$WORKDIR"
```

Tell the user where the report was saved.
