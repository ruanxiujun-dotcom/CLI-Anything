---
name: cli-hub-matrix-video-creation
description: >-
  Capability-based multi-tool matrix for video production. Agents pick providers
  (CLI-Anything harnesses, public CLIs, Python libs, native binaries, cloud APIs)
  per capability rather than marching through fixed stages, including storyboard
  planning, story/audio direction, internet video/music search/download,
  capture/generation, analysis, high-end caption design, review, and packaging.
---

# Video Creation Matrix (v3 — capability-based)

This matrix describes **capabilities** the agent can compose on demand — not a fixed pipeline. A "video creation" workflow picks a *recipe* (which capabilities it needs) and, per capability, picks a *provider* from the task requirements and preflight facts below.

Schema: [`docs/cli-matrix/matrix_registry.schema.md`](../../docs/cli-matrix/matrix_registry.schema.md).

## Install (installable portion)

```bash
cli-hub matrix install video-creation   # installs registered matrix CLIs only
cli-hub matrix info    video-creation   # inspect providers & recipes
cli-hub matrix preflight video-creation # check available providers in this environment
```

Not everything in this matrix is installed by `cli-hub matrix install`. Cloud APIs, Python packages, native binaries, third-party public CLIs, and external skills are first-class providers too, but install them only after the task actually needs that provider.
Do not hand-write `pip install ...#subdirectory=...` for CLI-Anything matrix members; install the supported harnesses through `cli-hub matrix install video-creation`, then use preflight to see what else is already available.

---

## Provider selection constraints (agent: evaluate per capability)

1. Use preflight as an availability report, not as a provider selector.
2. Choose providers from the user's goal, quality bar, budget, offline needs, credential state, install cost, and requested workflow.
3. Treat registry/provider order as documentation order only; do not assume the first provider is the correct one.
4. Install Python libs, native binaries, harness CLIs, public CLIs, or agent skills only when they fit the task constraints.
5. Escalate to paid or metered APIs only when the user has supplied credentials or explicitly consents. Never silently call a paid API.

Offline context? Filter to `offline: true` providers only.

---

## Preflight (run once per session, cache the result)

Run the built-in matrix preflight first:

```bash
cli-hub matrix preflight video-creation --json
cli-hub matrix preflight video-creation --capability composite.assemble
cli-hub matrix preflight video-creation --offline
```

If you need raw checks or are running without the latest `cli-hub`, use the manual block:

```bash
cli-hub list --json
python - <<'PY'
import importlib.util
for m in ("moviepy","whisper","pydub","PIL","edge_tts","pysrt","pysubs2","yt_dlp","spotdl","scenedetect","paddleocr","twelvelabs"):
    print(m, importlib.util.find_spec(m) is not None)
PY
for b in ffmpeg ffprobe sox convert magick screencapture yt-dlp spotdl scdl bandcamp-dl you-get lux BBDown scenedetect mediainfo ffmpeg-quality-metrics paddleocr hyperframes; do command -v "$b" >/dev/null && echo "$b: yes" || echo "$b: no"; done
for e in RUNWAY_API_KEY KLING_API_KEY PIKA_API_KEY SEEDANCE_API_KEY \
         ELEVENLABS_API_KEY MINIMAX_API_KEY OPENAI_API_KEY GOOGLE_CLOUD_PROJECT \
         ASSEMBLYAI_API_KEY DEEPGRAM_API_KEY \
         SUNO_API_KEY UDIO_API_KEY IDEOGRAM_API_KEY STABILITY_API_KEY \
         TWELVELABS_API_KEY GOOGLE_APPLICATION_CREDENTIALS; do
  [ -n "${!e}" ] && echo "$e: set" || echo "$e: unset"
done
```

---

## Suggest-to-user template (agent uses verbatim when escalating)

```
To enable <capability> via <provider>, please set <ENV_VAR>.
  Cost: <cost notes>
  Quality: <quality tier>
Reply 'skip' to fall back to <next provider>.
```

Examples:

- *To enable cinematic AI video via Runway Gen-4, please set `RUNWAY_API_KEY`. Cost: ~$0.05/sec as of 2026-04. Quality: sota. Reply 'skip' to fall back to `generate-veo-video` or `jimeng` if configured.*
- *To enable ByteDance Seedance video generation, please set `SEEDANCE_API_KEY`. Cost: metered per-clip. Quality: sota for realistic motion. Reply 'skip' to fall back to `jimeng` (Dreamina) which shares the ByteDance model family.*

---

## Capabilities

### `script.storyboard` — brief to creative direction, script, shot list, timing, and asset plan

Use this before generation, search, capture, or assembly when the user gives a vague concept or asks for a complete video. By default, do the planning directly as agent work: produce a structured brief, global creative direction, narrative/emotional arc, audio arc, script/narration, shot list, timing map, asset requirements, and reviewable storyboard before spending time on downloads or generation.

For any non-trivial video, read [`references/story-structure-audio.md`](references/story-structure-audio.md) and save `creative_direction.md` before final assembly. This is mandatory for trailers, sports/music montages, film commentary, found-footage edits, product launch videos, and any output where flat random clips or boring music would fail the brief.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| Agent-native planning | agent-native | none | free | high | yes |
| `storyboard-creation` skill | agent-skill | installed skill | free | high | yes |
| `remotion-best-practices` skill | agent-skill | installed skill | free | high for code-driven motion video | yes |

Selection:

- Start with agent-native planning for normal scripts, shot lists, timing maps, and asset plans.
- Include `creative_direction.md` with the one-sentence promise, story arc, emotional curve, audio arc, cut-density curve, visual motif, source roles, and no-flatness guardrails.
- Use `storyboard-creation` only when you need explicit storyboard-panel conventions, camera-angle grammar, continuity checks, or animatic planning.
- Use `remotion-best-practices` only when the storyboard will be implemented as Remotion/React motion-video code.

### `video.search` — discover candidate internet footage

Use this before `video.download` when the user asks for found footage, B-roll, public-domain clips, stock footage, YouTube/Bilibili material, or named movie/TV/game/anime moments. Prefer free/open sources first and record source URL, license, creator, and attribution requirement before editing.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| Web search + source filters | web-search | online search access | free | good | no |

Search discipline:

- For reusable/commercial-safe B-roll, start with Wikimedia Commons, YouTube Creative Commons, or other browser-searchable stock/public-domain pages and keep attribution metadata beside every downloaded file.
- For Bilibili/YouTube reference or fan-edit workflows, search specific scene names rather than generic terms. Add quality modifiers such as `1080p`, `4K`, `HD`, `BD`, `蓝光`, or `高清`.
- For Bilibili, use targeted web search such as `site:bilibili.com "Game of Thrones" S3E09 BV`; standalone `/video/BV...` uploads are usually easier to process than geo-restricted bangumi URLs.
- Do not treat downloadability as permission. If license or user authorization is unclear, ask before using the footage in a deliverable.

### `video.download` — download/import web video into the workspace

Use this after `video.search` identifies candidate URLs, or directly when the user gives URLs. Keep all raw downloads in one `sources/` directory, save `sources.json` with URL/license/creator/provenance, and normalize filenames before downstream editing.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `yt-dlp` | public-cli | `yt-dlp` + `ffmpeg` | free | high | no |
| `you-get` | public-cli | `you-get` bin | free | good | no |
| `lux` | public-cli | `lux` + `ffmpeg` | free | good | no |
| `BBDown` | public-cli | `BBDown` + `ffmpeg` | free | high for Bilibili | no |

Operational notes:

- Prefer `yt-dlp -f "bestvideo[height>=1080]+bestaudio/best" --merge-output-format mp4` for YouTube/Bilibili URLs when quality matters. Use cookies only when the user has authorized access to the content.
- For Bilibili audio-only extraction, avoid `yt-dlp -x --audio-format mp3`; download the raw m4a and convert with `ffmpeg`, then verify volume before using it.
- Use `BBDown` when Bilibili-specific metadata, subtitles, danmaku, playlists, or high-quality member streams are central to the task.

### `music.search` — discover existing songs, BGM, or clean audio sources

Use this before `music.download` when the user wants an existing song, soundtrack cue, royalty-free track, platform audio, or a specific cover/version. Keep the search about the requested music, not just any available audio. Record title, artist, platform URL, uploader, source type, version notes, rights/licensing, and attribution in `music_sources.json`.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| Web search + source filters | web-search | online search access | free | good | no |
| `yt-dlp` search extractors | public-cli | `yt-dlp` | free | good | no |
| `spotdl` metadata/search | public-cli | `spotdl` | free | good for Spotify-linked songs | no |

Search discipline:

- For clean audio, look for the artist's official upload, label upload, official audio/MV, verified lyric video, or a standalone audio upload with no extra descriptors.
- If the user asks for a specific version such as `女生版`, duet, piano, acoustic, live, DJ, karaoke, or instrumental, verify the title/uploader/metadata names that exact version before committing.
- Reject candidate titles that imply the wrong source: `remix`, `cover`, `fan edit`, `AMV`, `MAD`, `mashup`, compilation, trailer mix, or unrelated soundtrack/OST unless the user explicitly asked for that variant.
- For tie-in songs and promo tracks, assume dialogue/voiceover/SFX bleed is possible. Search alternatives with `{artist} {song} 纯音乐`, `{song} 无对白`, `{song} 歌词版`, `official audio`, or `lyric video` before asking the user to accept a risky source.
- Do not treat downloadability as permission. If rights, license, or user authorization is unclear, ask before using the music in a deliverable.

### `music.download` — download/import existing music into the workspace

Use this after `music.search` identifies a candidate or directly when the user supplies a music URL/local file. Keep raw downloads in `sources/music/`, save `music_sources.json`, and create a normalized working file for editing.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `yt-dlp` audio download | public-cli | `yt-dlp` + `ffmpeg` | free | high | no |
| `spotdl` | public-cli | `spotdl` + `ffmpeg` | free | good for Spotify-linked metadata | no |
| `scdl` | public-cli | `scdl` | free | good for SoundCloud | no |
| `bandcamp-dl` | public-cli | `bandcamp-dl` | free | good for Bandcamp | no |
| local file import + `ffmpeg` | native | `ffmpeg` | free | high | yes |

Operational notes:

- Use only music the user is authorized to use or that is clearly licensed for the deliverable. Do not bypass DRM, paywalls, or access controls.
- For general audio URLs, download source audio first, then convert explicitly:

```bash
yt-dlp -f "bestaudio[ext=m4a]/bestaudio" -o "sources/music/audio_raw.%(ext)s" "URL"
ffmpeg -i sources/music/audio_raw.m4a -vn -c:a libmp3lame -q:a 0 sources/music/audio.mp3
```

- For Bilibili audio-only downloads, do **not** use `yt-dlp -x --audio-format mp3`; it can produce a valid-looking but nearly silent MP3. Download the raw m4a, convert with `ffmpeg`, then verify volume:

```bash
yt-dlp -f 30280 -o "sources/music/audio_raw.%(ext)s" "BILIBILI_URL"
ffmpeg -i sources/music/audio_raw.m4a -c:a libmp3lame -q:a 0 sources/music/audio.mp3
ffmpeg -i sources/music/audio.mp3 -af volumedetect -f null - 2>&1 | grep mean_volume
```

Reject files with mean volume below roughly `-40dB` unless silence is expected.

- Sample-listen at least three points before editing, especially for promo/OST/tie-in songs:

```bash
ffplay -ss 30 -t 5 -autoexit sources/music/audio.mp3
ffplay -ss 90 -t 5 -autoexit sources/music/audio.mp3
ffplay -ss 150 -t 5 -autoexit sources/music/audio.mp3
```

- Normalize or loudness-match only after verifying the source is the right song/version and has no dialogue/SFX bleed.

### `visual.capture` — record screen / webcam / window

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `cli-anything-openscreen` | harness-cli | harness installed | free | high | yes |
| `cli-anything-obs-studio` | harness-cli | OBS installed | free | high | yes |
| `ffmpeg -f x11grab` / `avfoundation` | native | `ffmpeg` | free | high | yes |
| `screencapture` | native | macOS | free | high | yes |
| `mss` / `pyautogui` + `cv2` | python | pkgs | free | good | yes |

### `visual.generate` — produce a video clip from prompt/reference

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `generate-veo-video` | public-cli | `generate-veo` bin + Google creds | metered | high | no |
| `jimeng` | public-cli | `dreamina` bin + Dreamina login | metered | high | no |
| Runway Gen-4 | api | `RUNWAY_API_KEY` | paid | sota | no |
| Kling | api | `KLING_API_KEY` | paid | high | no |
| Pika | api | `PIKA_API_KEY` | paid | good | no |
| Seedance | api | `SEEDANCE_API_KEY` | paid | sota | no |

### `audio.capture` — record and clean audio tracks

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `cli-anything-audacity` | harness-cli | Audacity installed | free | high | yes |
| `sox` / `ffmpeg` | native | binary | free | high | yes |
| `pydub` / `soundfile` / `librosa` / `noisereduce` | python | pkgs | free | good | yes |

### `audio.synthesize` — text-to-speech / voice

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `minimax-cli` | public-cli | bin + MiniMax key | metered | high | no |
| `elevenlabs` | public-cli | bin + `ELEVENLABS_API_KEY` | paid | sota | no |
| OpenAI TTS | api | `OPENAI_API_KEY` | metered | high | no |
| Google Cloud TTS | api | `GOOGLE_CLOUD_PROJECT` | metered | high | no |
| `edge-tts` | python | pkg | free | good | no |

### `music.generate` — generated music / BGM

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `suno` | public-cli | bin + Suno account | metered | sota | no |
| `minimax-cli` | public-cli | bin + MiniMax key | metered | high | no |
| Udio | api | `UDIO_API_KEY` | paid | sota | no |

Use `music.search` + `music.download` instead when the user asks for an existing song, official upload, platform audio, soundtrack cue, royalty-free track, or a specific cover/version.

Music and SFX must follow the story/audio arc in `creative_direction.md`. For polished 60+ second videos, avoid one flat loop from start to finish; plan section changes such as intro, buildup, drop, dip, final lift, source-audio reveal, or final resolve.

### `media.analyze` — segment, label, OCR, and search footage

Use this after download/capture and before edit planning when there are many clips or when the edit depends on finding specific shots. Output should be a scene library with time ranges, keyframes, visible text, people/objects/actions where available, usability notes, and searchable tags.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| PySceneDetect `scenedetect` | public-cli | `scenedetect` + `ffmpeg` | free | high | yes |
| Google Cloud Video Intelligence | api | GCP creds | metered | sota | no |
| TwelveLabs video search/index | api | `TWELVELABS_API_KEY` + `twelvelabs` pkg | metered | sota | no |
| PaddleOCR on sampled keyframes | public-cli/python | `paddleocr` pkg or bin | free | good overall; high for visible text | yes |

Default path:

- Use PySceneDetect for general local cut detection and keyframe extraction.
- Use PaddleOCR only on sampled keyframes or subtitle regions; it complements scene detection rather than replacing it.
- Use Google Video Intelligence or TwelveLabs when you need high-quality object/person/action/text labels or semantic search over a larger footage library and the user accepts cloud processing/cost.

### `text.transcribe` — speech → text / subtitles

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `cli-anything-videocaptioner` | harness-cli | harness installed | free | high | yes |
| `openai-whisper` | python | pkg + model download | free | high | yes |
| `stable-ts` / `faster-whisper` | python | pkg + model download | free | high | yes |
| AssemblyAI | api | `ASSEMBLYAI_API_KEY` | paid | sota | no |
| Deepgram | api | `DEEPGRAM_API_KEY` | paid | sota | no |
| Google Speech-to-Text | api | GCP creds | metered | high | no |

Local ASR notes:

- Preflight package checks do not prove Whisper model weights are already cached or that runtime will fit the task.
- Prefer small/base CPU models for quick local drafts; use larger models only when the user accepts the time/resource tradeoff or the machine is known to handle it.
- Use paid/cloud ASR when long recordings, diarization, timestamps, or quality requirements make local model setup a poor fit.
- Do not use Whisper `.en` models unless the user explicitly says the audio is English; `.en` models translate non-English speech into English.

### `text.caption` — design, time, render, and QC visible captions

Use this after `text.transcribe` or agent-written script/narration timing, and before `composite.overlay` / `package.encode`. This is for viewer-facing subtitles, creator captions, trailer title hits, lyrics/karaoke, lower-third text, and other timed text that must look high-end. It is not just "burn SRT at the end."

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| Captions reference module | agent-native | `references/captions.md` | free | high | yes |
| ASS + `ffmpeg subtitles` | native | `ffmpeg` + fonts | free | high | yes |
| HyperFrames captions workflow | agent-skill | `npx skills add heygen-com/hyperframes --skill hyperframes`; Node.js + `ffmpeg` | free | high for kinetic/digital captions | yes |
| `pysubs2` ASS authoring | python | `pysubs2` pkg + `ffmpeg` | free | high | yes |
| MoviePy/Pillow transparent overlays | python | `moviepy` + `PIL` | free | good when custom layout is required | yes |

Caption discipline:

- Read [`references/captions.md`](references/captions.md) for any polished caption/subtitle/lyric/lower-third work. It defines the lifecycle, genre presets, typography, safe-zone rules, and QC rubric.
- Keep transcription (`text.transcribe`) separate from caption design (`text.caption`). A raw SRT is source material, not a finished caption package.
- Produce `captions.source.json`, `captions_style.md`, `captions.ass` or equivalent render source, caption-heavy preview frames/contact sheet, and `captions_qc.md` for non-trivial videos.
- Prefer ASS + `ffmpeg` for deterministic subtitle burn-in on edited footage. Use HyperFrames only when the captions are part of an HTML/CSS/GSAP motion composition, karaoke, audio-reactive typography, or digital launch video.
- Fail the render if captions are clipped, stale, off-sync, too small, low-contrast, covering faces/action/source subtitles, missing CJK glyphs, or visually mismatched to the genre.

### `composite.assemble` — timeline, cuts, transitions, export

For non-trivial videos, do not assemble a final timeline until `script.storyboard` has produced `creative_direction.md`. Timeline order should follow the story/audio arc and source roles from [`references/story-structure-audio.md`](references/story-structure-audio.md), not source-file order or arbitrary clip variety.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `cli-anything-kdenlive` | harness-cli | Kdenlive installed | free | high | yes |
| `cli-anything-shotcut` | harness-cli | Shotcut installed | free | high | yes |
| `moviepy` | python | pkg + `ffmpeg` | free | good | yes |
| `ffmpeg-python` | python | pkg + `ffmpeg` | free | high | yes |
| `ffmpeg concat/filter_complex` | native | `ffmpeg` | free | high | yes |
| HyperFrames skill/CLI | agent-skill | `npx skills add heygen-com/hyperframes`; Node.js + `ffmpeg` | free | good for digital launch videos | yes |

Use HyperFrames only for website/product-page or digital UI-driven launch videos where an HTML/CSS/GSAP composition is the right authoring surface. Use `remotion-best-practices` instead when the implementation must stay in Remotion/React.

Assembly discipline:

- Every selected shot needs a job: setup, reveal, escalation, contrast, proof, impact, or payoff.
- Every 10-20 seconds, something should change: tension, cut density, audio texture, title system, setting, character focus, or edit pattern.
- Chapter cards and title hits must advance the story beat; they cannot be decorative separators for random clips.
- If a render is technically valid but feels like flat montage, revise `creative_direction.md` and rebuild the timeline instead of adding more filters.

### `composite.overlay` — composite captions, watermark, picture-in-picture

Use this to apply already-designed caption/subtitle assets, watermarks, or picture-in-picture layers. For user-visible subtitles/captions, run `text.caption` first so the overlay step receives a deliberate ASS/HTML/PNG/NLE caption package instead of an ugly default SRT.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `ffmpeg -vf subtitles=...` | native | `ffmpeg` | free | high | yes |
| `moviepy` (CompositeVideoClip) | python | pkg | free | good | yes |

### `package.thumbnail` — thumbnail / social card

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `cli-anything-gimp` / `krita` / `inkscape` | harness-cli | installed | free | high | yes |
| `Pillow` | python | pkg | free | good | yes |
| `cairosvg` / `html2image` | python | pkg | free | good | yes |
| OpenAI GPT-Image-1 | api | `OPENAI_API_KEY` | metered | sota | no |
| Google Nano Banana | api | GCP creds | metered | high | no |
| Ideogram | api | `IDEOGRAM_API_KEY` | metered | high | no |
| Stability AI | api | `STABILITY_API_KEY` | metered | high | no |
| `ffmpeg -ss ... -frames:v 1` | native | `ffmpeg` | free | basic | yes |
| `convert` / `magick` | native | ImageMagick | free | good | yes |

### `package.encode` — final mux, codec, container

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `ffmpeg` | native | binary | free | sota | yes |

### `quality.review` — technical and editorial QC before delivery

Run this after every render and before presenting a final file. Fail loudly on black frames, silence, loudness outliers, wrong dimensions/aspect ratio, bad bitrate/container, subtitle/layout problems, repeated shots, flash frames at cut heads, flat story structure, or boring/unchanging audio.

| Provider | Kind | Requires | Cost | Quality | Offline |
|---|---|---|---|---|---|
| `ffmpeg` / `ffprobe` QC filters | native | `ffmpeg` + `ffprobe` | free | high | yes |
| MediaInfo CLI | native | `mediainfo` | free | high | yes |
| `ffmpeg-quality-metrics` / VMAF | public-cli | `ffmpeg-quality-metrics` + `ffmpeg` | free | high with reference | yes |

QC checks:

- `ffprobe -of json -show_streams -show_format` for duration, codec, bitrate, width, height, frame rate, and aspect ratio.
- FFmpeg filters: `blackdetect`, `silencedetect`, `freezedetect`, `cropdetect`, `ebur128` / `loudnorm`.
- Extract first/mid/last frames and subtitle-heavy frames; inspect visually or via OCR for text overflow, watermarks, black frames, and wrong crops.
- Use VMAF/SSIM/PSNR only when a reference video exists; they are not general no-reference quality checks.
- For non-trivial videos, run the story/audio review rubric in [`references/story-structure-audio.md`](references/story-structure-audio.md): premise clarity, escalation, turning point, high point, source roles, music sections, audio events, and final payoff.

### `publish.upload` — deliver to a platform

Currently a **known gap** (see below). Agents should surface this to the user.

---

## Recipes

Recipes declare *which capabilities a workflow needs* — not the order. Choose providers per capability from the task constraints and preflight facts.

- **`ai-short`** — fully-generative social short.
  Uses: `script.storyboard`, `visual.generate`, `audio.synthesize`, `music.generate` or `music.search` / `music.download`, `text.caption` when captions/title hits are part of the brief, `composite.assemble`, `composite.overlay`, `package.thumbnail`, `quality.review`, `package.encode`.

- **`screencast-tutorial`** — walkthrough with narration + subs.
  Uses: `script.storyboard`, `visual.capture`, `audio.capture`, `text.transcribe`, `text.caption`, `composite.overlay`, `package.thumbnail`, `quality.review`, `package.encode`.

- **`talking-head-explainer`** — webcam + b-roll + captions.
  Uses: `script.storyboard`, `visual.capture`, `video.search` / `video.download` or `visual.generate` (b-roll), `music.search` / `music.download` or `music.generate`, `media.analyze`, `audio.capture`, `text.transcribe`, `text.caption`, `composite.assemble`, `composite.overlay`, `package.thumbnail`, `quality.review`, `package.encode`.

- **`podcast-to-video`** — audio-first, visualize + caption.
  Uses: `script.storyboard`, `audio.capture`, `text.transcribe`, `text.caption`, `package.thumbnail`, `composite.overlay`, `composite.assemble`, `quality.review`, `package.encode`.

- **`found-footage-montage`** — source internet clips, curate, then edit.
  Uses: `script.storyboard`, `video.search`, `video.download`, `media.analyze`, `text.transcribe` (optional), `text.caption` when captions/title hits are part of the edit, `composite.assemble`, `composite.overlay`, `package.thumbnail`, `quality.review`, `package.encode`.

- **`existing-song-music-video`** — build an edit around a user-specified or discovered song.
  Uses: `music.search`, `music.download`, `script.storyboard`, `video.search`, `video.download`, `media.analyze`, `text.caption` (optional lyrics/karaoke), `composite.assemble`, `composite.overlay`, `package.thumbnail`, `quality.review`, `package.encode`.

- **`digital-product-launch`** — product/site-driven launch video with animated UI, typography, and motion graphics.
  Uses: `script.storyboard`, `visual.capture`, `audio.synthesize` (optional), `music.generate` or `music.search` / `music.download`, `text.caption` for brand-safe kinetic captions/title hits, `composite.assemble` (HyperFrames only when the HTML/GSAP launch-video workflow matches), `composite.overlay`, `package.thumbnail`, `package.encode`, `quality.review`.

---

## Known gaps

- **`publish.upload`** — no first-party or public CLI for YouTube/TikTok/Bilibili/Instagram yet. *Workaround:* instruct the user to upload manually via the web UI, or escalate to a custom script using each platform's v3 API with an OAuth token the user supplies.
- **`visual.generate` — top-tier "cinematic"** — available only via paid APIs (Runway, Kling, Seedance); local GPU/weights deployment is intentionally not part of this matrix.
- **`rights.provenance`** — no automated license/TOS/provenance verifier. *Workaround:* save `sources.json` / `music_sources.json` with URL, creator, license, intended use, and attribution text; ask the user before using unclear or restricted media.
- **`agent-skill.preflight`** — external agent skills may appear in this matrix before they are installed locally. *Workaround:* use the source/install table below and load the external `SKILL.md` only when that workflow is actually needed.

---

## Reference modules, external skills, and tool sources

Consult these only when the task needs the focused workflow; keep this matrix as the router and load the reference, external skill, or tool only if its workflow matches.

| Reference/tool | Use when | Source or install |
|---|---|---|
| `advanced-video-downloader` | User supplies YouTube/Bilibili/TikTok/etc. URLs and needs download, playlist handling, music/audio extraction, cookies, or transcription | `npx skills add https://github.com/jst-well-dan/skill-box --skill advanced-video-downloader` |
| `references/story-structure-audio.md` | Non-trivial video needs a global arc, internal logic, source roles, story ups/downs, music/audio sections, or flat-montage prevention | Local reference module: [`references/story-structure-audio.md`](references/story-structure-audio.md) |
| `references/captions.md` | Any polished visible captions, subtitles, lyrics, karaoke, lower thirds, trailer title hits, or caption QC | Local reference module: [`references/captions.md`](references/captions.md) |
| `storyboard-creation` | Need shot grammar, camera angles, storyboard panels, continuity, or animatic planning | `pnpm dlx add-skill https://github.com/inference-sh/skills/tree/HEAD/guides/video/storyboard-creation` |
| `remotion-best-practices` | Need to implement the storyboard as Remotion code | `npx skills add https://github.com/remotion-dev/skills --skill remotion-best-practices`; SKILL.md: <https://github.com/remotion-dev/skills/blob/main/skills/remotion/SKILL.md> |
| `hyperframes` | Website/product-page or digital UI-driven launch videos where HTML/CSS/GSAP composition and local render are appropriate; also useful for kinetic synced captions when the whole video is HTML/GSAP-driven | `npx skills add heygen-com/hyperframes --skill hyperframes`; captions reference: <https://skills.sh/heygen-com/hyperframes/captions> |
| `ffmpeg-quality-metrics` | Need VMAF/SSIM/PSNR against a reference video | `pipx install ffmpeg-quality-metrics`; source: <https://github.com/slhck/ffmpeg-quality-metrics> |

---

## Agent guidance

- **Run the preflight block once**, then consult the cached result when picking providers.
- **For non-trivial videos, start with `creative_direction.md`.** Use the story/audio reference module to define the whole arc before acquisition and assembly: premise, emotional curve, audio arc, cut-density curve, source roles, turning point, climax, and payoff.
- **For internet footage, run `video.search` before `video.download`** unless the user already supplied URLs. Keep provenance metadata with every source file.
- **For existing music, run `music.search` before `music.download`** unless the user already supplied a URL/local file. Verify the song version and listen for dialogue/SFX bleed before beat analysis or editing.
- **For visible captions, use `text.caption` as its own design pass.** Do not hand off raw SRT to `composite.overlay` and call it done. Use the captions reference module for genre style, grouping, safe placement, font/glyph checks, hard exits, and caption-heavy preview frames.
- **For subtitle/caption localization, translate directly as agent work** unless the user explicitly requires a specific translation service, offline translation, or human/legal review. Preserve timestamps, maintain names/terms consistently, and verify CJK glyph rendering in overlays.
- **For non-trivial work, do `script.storyboard` before acquisition/generation**, usually as agent-native planning. Load an external storyboard skill only when its focused format is worth the extra install/context. Do not replace the global story/audio plan with a list of clips.
- **For digital product/site launch videos, consider HyperFrames under `composite.assemble`** only when the output should be an animated HTML/CSS/GSAP motion piece. Do not treat it as a general NLE replacement or use it for user requests that explicitly require Remotion/React.
- **Use Remotion guidance only for code-driven motion graphics.** Do not install it for ordinary cuts, overlays, or NLE work.
- **Choose Shotcut, Kdenlive, MoviePy, or raw `ffmpeg` by edit complexity and availability.** NLE harnesses fit timeline-heavy edits; MoviePy/ffmpeg fit deterministic programmatic cuts and overlays.
- **Prefer `--json`** for harness CLI output when chaining tools.
- **Escalate explicitly.** When a paid API would materially improve quality, use the suggest-to-user template. Do not silently burn credits.
- **Recipes ≠ order.** A recipe says what's needed; pick a sensible order for the specific task. Most videos should transcribe *after* the final cut, not before; screencasts often capture audio + video simultaneously.
- **Workspace discipline.** Keep all intermediate assets under one directory so cross-tool references stay stable.
