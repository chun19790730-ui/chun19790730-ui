# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repository stores custom Claude Code skills (slash commands). Currently it contains one skill: `/영상`, a Korean-language pipeline for adding hardcoded subtitles to video files.

## Repository Structure

```
.claude/commands/   # Custom slash-command skill definitions (Markdown)
.gitignore          # Excludes large media files and sensitive credentials
```

Each `.md` file in `.claude/commands/` defines one slash command — the filename becomes the command name (e.g. `영상.md` → `/영상`).

## The `/영상` Skill

**Trigger**: `/영상 [영상파일경로]`

**Pipeline** (run in this order):

1. **Whisper** — transcribes audio to `.srt` subtitle file
2. **Manual review** — user edits timing/text in the `.srt`
3. **Pillow hardcode** — extracts frames via ffmpeg, stamps subtitles per-frame with `PIL.ImageDraw`, reassembles with audio
4. **YouTube upload** (optional) — via YouTube Data API v3

**Required system dependencies:**

```bash
brew install ffmpeg
pip3 install openai-whisper Pillow
```

**Optional (YouTube upload):**

```bash
pip3 install google-api-python-client google-auth-oauthlib
# Also requires client_secrets.json from Google Cloud (YouTube Data API v3)
```

**Key implementation details in the Pillow hardcode step:**

- ffmpeg path is hardcoded to `/opt/homebrew/bin/ffmpeg` (macOS/Homebrew)
- Korean font path: `/System/Library/Fonts/AppleSDGothicNeo.ttc` (macOS system font)
- Subtitle stroke is drawn by rendering black text at ±2px offsets before the white fill — this is the outline effect, not a proper stroke API
- Frames are written to `/tmp/frames/` and can consume hundreds of MB depending on video length
- The Pillow approach is the fallback for when ffmpeg is compiled without `libass`/`freetype`

**Whisper model trade-offs:**

| Model | Size | Korean accuracy |
|-------|------|-----------------|
| small | ~461 MB | lowest |
| medium | ~1.5 GB | better |
| large | ~3 GB | best |

## Adding New Skills

Create a new `.md` file in `.claude/commands/`. The filename (without extension) becomes the slash command name. Follow the existing skill format: describe usage, then provide step-by-step instructions with runnable code blocks.

## gitignore Conventions

The `.gitignore` excludes all common video/image/PDF formats and `client_secrets.json`/`youtube_token.pickle`. Never commit credentials or large media files to this repo.
