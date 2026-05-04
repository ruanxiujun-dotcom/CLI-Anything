# Video Captions

Use this reference when captions are part of the viewer-facing video design, not just an accessibility sidecar. In the video-creation matrix, it sits after `text.transcribe` or agent-written script work and before `composite.overlay`, `package.encode`, and `quality.review`.

Default subtitle output is not acceptable for polished videos. Captions must feel designed for the video genre, synchronized to the edit, readable on the target platform, and visually integrated with the footage.

## Workflow

1. **Define the caption job.** Record aspect ratio, delivery platform, language(s), transcript source, whether word timings are available, and caption role: accessibility subtitles, creator captions, film commentary, trailer title hits, lyrics/karaoke, tutorial labels, or product launch typography.
2. **Build a clean timed source.** Produce `captions.source.json` with text, start, end, optional words, speaker, role, and emphasis. Use `.srt` only as an interchange format; keep JSON/ASS as the design source of truth.
3. **Select a style preset.** Pick from the genre presets below, then adapt to the footage palette, subject position, and music/edit energy. Do not use generic white text with a black stroke unless the user explicitly asks for plain subtitles.
4. **Design safe placement.** Extract representative frames for caption-heavy moments, mark face/action/logo/source-subtitle regions, and choose one or two safe zones. Captions must not cover faces, important action, hardcoded subtitles, UI controls, or source watermarks.
5. **Render with the right authoring path.** Use ASS+ffmpeg for deterministic subtitles, HyperFrames/HTML for kinetic digital captions, NLE overlay tracks for timeline-heavy edits, or transparent PNG/MoviePy overlays only when custom layout is necessary.
6. **QC before delivery.** Save `captions_qc.md` and preview frames/contact sheets. Fail the render if captions are clipped, stale, too small, low contrast, off-sync, visually cheap, or mismatched to the video genre.

## Transcript Rules

- Never use Whisper `.en` models unless the user explicitly says the audio is English. `.en` models translate non-English audio into English instead of transcribing it.
- If the user gives a script, time captions against the final narration/audio, not against the draft text.
- If translating/localizing captions, do the translation as agent work unless the user requires a specific service. Preserve timestamps, names, terms, tone, and line breaks; then QC glyph rendering.
- Keep one caption group visible at a time for spoken captions. Trailer title hits and lower thirds may coexist only if they occupy distinct zones and do not compete for reading priority.
- Break groups on sentence boundaries, semantic phrases, beat hits, or pauses longer than about 150 ms.

## Deliverables

Every captioned video should keep these files near the render:

- `captions.source.json` — timed text source with role/emphasis metadata.
- `captions_style.md` — selected preset, font, palette, placement, animation, and deviations.
- `captions.ass` or equivalent render source; optionally `captions.srt` for accessibility/export.
- `captions_preview_frames/` or `review_frames/` — first/middle/last and caption-dense samples.
- `captions_qc.md` — readability, sync, overflow, safe-zone, font, and style review.

## Style Presets

### Sports Hype / Music Montage

- **Use for:** dunk reels, football/soccer hype, workout edits, esports frag montages.
- **Text density:** 1-3 words per hit; avoid paragraph subtitles.
- **Typography:** heavy condensed sans, 800-900 weight; CJK needs a bold high-legibility font such as Noto Sans CJK/Source Han Sans.
- **Look:** dark or neutral caption plate, one aggressive accent, subtle glow or shadow, no thick default outline.
- **Motion:** beat-locked slam, scale-pop, wipe, quick flash, or kinetic word replacement. Use fast exits and hard kills.
- **Placement:** lower third for readable captions; center or upper-center only for short title hits between action beats.

### Film Commentary / Recap

- **Use for:** movie explanation, anime recap, documentary narration, story analysis.
- **Text density:** 6-14 Chinese characters per line or 28-40 Latin characters per line; max two lines.
- **Typography:** elegant sans or restrained serif for chapter cards; normal captions should stay highly readable.
- **Look:** cinematic neutral/warm palette, soft shadow, thin accent line or small chapter label, no loud bouncing.
- **Motion:** gentle fade/slide for narration; stronger title cards only at chapter boundaries.
- **Placement:** bottom safe zone unless source subtitles or important faces occupy it; use upper safe zone only when necessary.

### Sci-Fi / Game / Mecha Trailer

- **Use for:** CG trailers, game launch edits, anime/mecha/sci-fi hype.
- **Text density:** short bilingual title hits, faction labels, mission-style captions, 2-4 word bursts.
- **Typography:** square/tech sans, condensed bold, or clean mono for data-like labels; keep letter spacing at 0 for body captions.
- **Look:** high-contrast white/near-white plus one neon accent; restrained glow, scanline, bracket, or HUD framing.
- **Motion:** glitch, scan reveal, mask wipe, chromatic nudge, or hard cut on impact. Avoid random particles and cheap typewriter spam.
- **Placement:** title hits can use center frame during low-action beats; subtitles stay bottom/side safe and never cover spectacle.

### Product / Digital Launch

- **Use for:** website-to-video, app demos, SaaS launches, Remotion/HyperFrames motion pieces.
- **Text density:** short product claims, feature labels, metric callouts, step captions.
- **Typography:** match `design.md` or product brand fonts. If missing, use a modern sans with consistent weights.
- **Look:** brand palette, clean contrast, precise spacing, polished cards/chips only when the product UI style supports them.
- **Motion:** layout-first kinetic type, marker sweeps, reveal masks, scroll/pointer sync, audio-reactive emphasis if it supports the beat.
- **Implementation:** HyperFrames is appropriate when the video is HTML/CSS/GSAP driven. Install with `npx skills add heygen-com/hyperframes --skill hyperframes` and read its captions guidance before authoring kinetic synced text.

### Tutorial / Explainer

- **Use for:** screencasts, app walkthroughs, educational clips, code demos.
- **Text density:** clear phrases; prefer fewer captions when screen text already carries meaning.
- **Typography:** clean sans; mono only for code terms. Use exact UI labels and command names.
- **Look:** quiet, high contrast, no decorative motion that distracts from the action.
- **Motion:** quick fade/slide, pointer-aligned callouts, occasional highlight boxes.
- **Placement:** avoid covering cursor targets, terminal prompts, menus, code, or UI labels.

### Lyrics / Karaoke

- **Use for:** music videos, lyric edits, singalong, rhythm shorts.
- **Text density:** phrase or word-level timing; lyrics should follow musical phrasing, not sentence grammar.
- **Typography:** genre-matched display font for hooks, readable sans for verses.
- **Look:** one active-word treatment plus one base caption style. Avoid rainbow karaoke unless the user asks for it.
- **Motion:** per-word color fill, underline sweep, scale emphasis, or mask reveal. Keep sync tighter than normal speech captions.
- **Placement:** bottom or center-lower; make sure fast cuts do not leave stale lyrics from the previous phrase.

### Vertical Social / Talking Head

- **Use for:** shorts, Reels, TikTok-style hooks, selfie explainers.
- **Text density:** 2-5 words per group; emphasize hooks, numbers, names, and claims.
- **Typography:** bold rounded or bold grotesk, large enough for phone viewing.
- **Look:** strong foreground/background contrast, restrained pill/plate, one accent color for keywords.
- **Motion:** pop/slide per group, occasional keyword bounce; avoid constant elastic motion.
- **Placement:** lower-middle or mid-lower safe zone, but never over mouth/face. Leave platform UI margins.

## Typography And Layout

- Size for final pixels, not editor preview. For 16:9 1080p, spoken captions usually land around 48-76 px; hype/title hits can be 80-130 px. For vertical 1080x1920, spoken groups usually need 64-104 px.
- Use max width: about 70-82% of landscape width, 78-88% of portrait width. Reduce width further when words scale above 1.0.
- Set explicit line height around 1.05-1.18. Do not rely on default browser or ASS line spacing.
- Use real safe margins: at least 5% from frame edges, more for vertical social platform UI.
- Verify fonts render all glyphs. CJK tofu boxes, missing punctuation, broken emoji, or fallback font jumps are hard failures.
- Prefer shadow, blur, backing plate, or glow tuned to the footage over a thick black outline. If using ASS outlines, keep them intentional and proportional.
- Keep body caption letter spacing at 0. Avoid negative tracking. Display title cards may use deliberate tracking only if it improves the genre look.

## Authoring Paths

### ASS + FFmpeg

Use when the final output is normal edited footage and captions need deterministic burn-in.

```bash
ffmpeg -i input.mp4 -vf "subtitles=captions.ass:fontsdir=fonts" -c:a copy output_captioned.mp4
```

ASS is the preferred interchange for styled subtitles because it supports font, size, outline, shadow, position, and per-event timing. Keep `captions.ass` readable and style-named (`Narration`, `Keyword`, `Chapter`, `LyricActive`) instead of generating anonymous styles.

### HyperFrames / HTML Captions

Use when captions are part of a digital motion composition, product launch, audio-reactive typography, karaoke, or per-word kinetic text system.

- Install/load only when the workflow needs it: `npx skills add heygen-com/hyperframes --skill hyperframes`.
- Build end-state layout first, then animate into/out of that layout.
- Use deterministic timelines, no random/time-based logic, and no infinite repeats.
- Use fit-to-width logic for dynamic text and a hard timeline kill at each group end so old captions cannot remain visible.

### NLE Overlay Tracks

Use Shotcut/Kdenlive when the whole edit already lives in an NLE timeline. For complex captions, pre-render transparent overlays or ASS-burned intermediate clips, then bring them into the NLE to avoid fragile text filter behavior.

### Python / MoviePy / Pillow

Use only when the project already uses a Python render path and needs custom layout that ASS cannot express. Render transparent text layers at final resolution, inspect frames, and avoid rebuilding a low-end subtitle engine from scratch.

## QC Rubric

A caption pass fails if any of these are true:

- Captions feel like generic subtitles pasted on top of the video.
- Any caption is clipped, outside safe margins, too small on the target platform, or unreadable against the footage.
- Captions cover faces, important action, source subtitles, UI controls, or watermarks without a documented reason.
- Text lingers past its end time, overlaps the next caption group unintentionally, or appears before the spoken line.
- CJK/Latin font fallback is inconsistent, missing glyphs appear, or punctuation wraps badly.
- Style conflicts with genre: bouncy social captions on serious film commentary, plain subtitles on a hype montage, loud karaoke on a tutorial, etc.
- The final render was delivered without caption-heavy sample frames or a written caption QC note.

Minimum QC steps:

```bash
ffprobe -v error -show_streams -show_format -of json final.mp4 > final_probe.json
ffmpeg -y -i final.mp4 -vf "select='not(mod(n,120))',scale=480:-1,tile=5x4" -frames:v 1 captions_contact_sheet.jpg
```

Also extract exact frames around first caption, densest caption section, last caption, and any style transition. Inspect them visually before calling the video done.
