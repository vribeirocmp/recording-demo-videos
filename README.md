<div align="center">

# 🎬 recording-demo-videos

**A Claude Code skill that turns "record a demo of my app" into a scripted, captioned — and optionally AI-narrated — mp4.**

No screen recorder. No microphone. No video editor. Re-record in one command whenever your product changes.

[![Claude Code skill](https://img.shields.io/badge/Claude%20Code-skill%20%2B%20plugin-D97757?logo=anthropic&logoColor=white)](#-install)
[![Playwright](https://img.shields.io/badge/Playwright-recording-2EAD33?logo=playwright&logoColor=white)](https://playwright.dev)
[![Voice: Kokoro TTS](https://img.shields.io/badge/voice-Kokoro%20TTS%20(offline)-6E56CF)](https://huggingface.co/hexgrad/Kokoro-82M)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Buy me a beer](https://img.shields.io/badge/Buy%20me%20a%20beer-%F0%9F%8D%BA-FFDD00?logo=buymeacoffee&logoColor=black)](https://buymeacoffee.com/xh1d3xj)

🇺🇸 English · [🇧🇷 Português](README.pt-BR.md)

<a href="assets/demo-en.mp4"><img src="assets/demo-preview.gif" alt="Demo video recorded and narrated by this skill" width="720"></a>

**▶️ [Watch it with sound (English)](assets/demo-en.mp4)** · [🇧🇷 Portuguese version](assets/demo-pt.mp4)<br>
<sub>This video was scripted, recorded and narrated by the skill itself — see <a href="examples/sample-app/record-demo.mjs">examples/sample-app</a>, recorded against a small fictional app included in this repo.</sub>

</div>

---

## ✨ What you get

| | |
|---|---|
| 🎭 **A pitch, not a screencast** | Claude writes a 4-act script (vision → how it works → user experience → proof) where every scene proves one point. |
| 🤖 **Real interactions** | Playwright drives a real browser: visible typing, clicks, navigation, persona switches. |
| 💬 **On-screen captions** | Branded caption bar with a headline + subtext on every scene. |
| 🟧 **Guided attention** | An amber highlight appears on each element before it's clicked. |
| 🎙️ **AI voice-over** *(optional)* | Natural English narration from [Kokoro](https://huggingface.co/hexgrad/Kokoro-82M) — **offline**, runs on CPU, Apache-2.0. Brazilian Portuguese supported with Microsoft neural voices (online) or Kokoro (offline). |
| ⏱️ **Audio-driven pacing** | Each scene lasts exactly as long as its narration — the voice is never cut off and lines never overlap. |
| 🎯 **Automatic A/V sync** | A sync marker measures Playwright's (variable) recording delay and aligns every line to its caption within ~0.1 s. |
| 📦 **Ready to share** | `mp4` (H.264 + AAC — plays in PowerPoint, Slack, WhatsApp), `.srt` subtitles, and a re-runnable script (`npm run demo:record`). |
| ✅ **Verified, not hoped** | The skill makes Claude inspect frames and check voice timing before calling the video done. |

## 🚀 Usage

Ask Claude Code in plain words — *"record a demo video of our checkout flow"* — or use the slash command:

| Command | Result |
|---|---|
| `/recording-demo-videos` | Silent video with captions *(original behavior)* |
| `/recording-demo-videos audio` | 🇺🇸 English narration, offline Kokoro voice |
| `/recording-demo-videos audio pt` | 🇧🇷 Brazilian Portuguese narration (edge voice; offline Kokoro fallback) |
| `/recording-demo-videos audio pt kokoro` | 🇧🇷 Portuguese, fully offline |
| `/recording-demo-videos audio en edge` | English with Microsoft neural voices via `edge-tts` (online — see [voices](#-voices)) |

> Installed as a plugin? The command is namespaced: `/recording-demo-videos:recording-demo-videos audio`.

Claude then writes the script with you, adapts the template to your app (selectors, routes, brand colors), records, and verifies the result.

## 📥 Install

### Option A — Claude Code plugin (recommended)

```text
/plugin marketplace add https://github.com/vribeirocmp/recording-demo-videos
/plugin install recording-demo-videos@vribeirocmp
```

### Option B — Copy the skill (shortest slash command)

```bash
git clone https://github.com/vribeirocmp/recording-demo-videos
# macOS / Linux
cp -r recording-demo-videos/skills/recording-demo-videos ~/.claude/skills/
# Windows (PowerShell)
Copy-Item -Recurse recording-demo-videos\skills\recording-demo-videos $HOME\.claude\skills\
```

### Requirements

| Tool | Needed for | Install |
|---|---|---|
| [Node.js](https://nodejs.org) 20.11+ | the recording script | — |
| [Playwright](https://playwright.dev) | browser automation + capture | `npm i -D playwright && npx playwright install chromium` |
| [ffmpeg](https://ffmpeg.org) | mp4 conversion, audio mixing | `winget install Gyan.FFmpeg` · `brew install ffmpeg` · `apt install ffmpeg` |
| [uv](https://docs.astral.sh/uv/) *(narrated mode only)* | runs the TTS helper with zero setup | `pip install uv` or see docs |

The first narrated run downloads the Kokoro model once (~120 MB, cached in `~/.cache/recording-demo-videos`). After that, narration works fully offline.

## 🗣️ Voices

| Engine | Language | Default voice | Other voices | Notes |
|---|---|---|---|---|
| **Kokoro** *(default for English)* | 🇺🇸 English | `af_heart` | `am_michael`, `af_bella`, `bf_emma` 🇬🇧, `bm_george` 🇬🇧 … | Offline, CPU, Apache-2.0. Excellent English. |
| **Kokoro** | 🇧🇷 Portuguese | `pf_dora` | `pm_alex`, `pm_santa` | Offline, but noticeably less natural than English — use when you can't go online. |
| **edge** | 🇺🇸 English | `en-US-AvaMultilingualNeural` | `en-US-AndrewMultilingualNeural` … | Best quality, **needs internet**. |
| **edge** *(default for Portuguese)* | 🇧🇷 Portuguese | `pt-BR-FranciscaNeural` | `pt-BR-AntonioNeural` … | Best Portuguese quality, **needs internet** (falls back to Kokoro offline). |

Pick a voice with `DEMO_VOICE=am_michael` and an engine with `DEMO_TTS=kokoro|edge` (default `auto`: English → Kokoro, Portuguese → edge). Full Kokoro list: [VOICES.md](https://huggingface.co/hexgrad/Kokoro-82M/blob/main/VOICES.md).

> ⚠️ `edge` uses [edge-tts](https://github.com/rany2/edge-tts), which talks to an **unofficial** Microsoft endpoint. Great for internal demos and drafts; for commercial use prefer Kokoro or an official TTS service.

## ⚙️ How it works

```text
 SCRIPT (caption + say per scene)
        │
        ▼
 1. narrate.py ── Kokoro / edge ──► one audio clip per scene (+ durations, cached)
        │
        ▼
 2. Playwright records the browser
      • sync marker: animated pre-roll → magenta flash (clock time noted)
      • narrate(page, "scene") → shows caption, notes the time, waits for the line to end
        │
        ▼
 3. ffmpeg
      • finds the flash in the video → exact recording offset → trims the flash
      • places each clip at its scene's time (adelay + amix + loudnorm)
      • ► demo.mp4 (narrated) + demo.srt + timeline.json
```

Why the sync marker? Playwright's video starts **0.2–1.5 s after the page opens**, and the delay changes between runs and versions. A fixed offset drifts; measuring a known visual event in the recording doesn't.

## 📁 What's inside

```text
skills/recording-demo-videos/
├── SKILL.md                         # the instructions Claude follows (mode switch, process, pitfalls)
├── record-demo-template.mjs         # silent template: captions, highlights, login, mp4
├── record-demo-narrated-template.mjs# narrated template: TTS + audio-driven pacing + auto-sync
└── narrate.py                       # per-scene TTS helper (uv run, Kokoro / edge, cached)
examples/sample-app/                 # fictional "Launch Checklist" app + the script that recorded the video above
.claude-plugin/                      # plugin + marketplace manifests
```

## 🧪 Try the example

```bash
git clone https://github.com/vribeirocmp/recording-demo-videos && cd recording-demo-videos
npm install && npx playwright install chromium
npm run example                                        # English → examples/sample-app/out/demo.mp4
DEMO_LANG=pt node examples/sample-app/record-demo.mjs   # Portuguese
```

## 🩺 Troubleshooting

| Symptom | Fix |
|---|---|
| `Sync marker not found` in the log | Keep `syncMarker(page)` as the very first action, then re-record. |
| White/blank screens in the video | Run your app in production mode (or warm up dev routes) and wait for `networkidle`. |
| Voice mispronounces an acronym | Spell it phonetically in `say` (`"S Q L"`); the caption keeps the real text. |
| `uv` not found | Install it (`pip install uv`) — only needed for narrated mode. |
| mp4 won't play in PowerPoint | Already handled: H.264 `yuv420p` + AAC. Check you're sharing `demo.mp4`, not `demo.raw.webm`. |

## 🙏 Credits

Built on the shoulders of [Playwright](https://playwright.dev), [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) (via [kokoro-onnx](https://github.com/thewh1teagle/kokoro-onnx)), [edge-tts](https://github.com/rany2/edge-tts), [FFmpeg](https://ffmpeg.org) and [uv](https://docs.astral.sh/uv/).

Contributions welcome — issues and PRs with new voices, languages or pitfalls you hit are especially appreciated.

## 🍺 Support

If this skill saved you an afternoon of screen recording, consider buying me a beer:

<a href="https://buymeacoffee.com/xh1d3xj"><img src="https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20beer&emoji=%F0%9F%8D%BA&slug=xh1d3xj&button_colour=FFDD00&font_colour=000000&font_family=Cookie&outline_colour=000000&coffee_colour=ffffff" alt="Buy me a beer" height="48"></a>

## 📄 License

[MIT](LICENSE) © Vinicius Ribeiro
