# Video Creation Matrix · YouTube/Bilibili High-Bar Challenge Briefs

Use this file to test [`cli-hub-matrix/video-creation/SKILL.md`](../../../cli-hub-matrix/video-creation/SKILL.md) with a fresh agent. This is a high-bar creative editing test, not a rendering demo. The outputs should feel like current high-quality short-form edits: sharp source footage, strong pacing, intentional typography, aggressive music timing, polished transitions, and visible editing taste.

For this evaluation, the videos are local learning/reference outputs. The agent should use real, high-quality YouTube and Bilibili videos as primary source material. Do not fall back to Pexels/Pixabay stock, static slides, generated placeholder footage, OpenCV/Pillow drawings, random neon UI, or "good enough" B-roll unless the task explicitly asks for it.

Do not mark a task complete if it looks cheap, generic, slow, under-edited, visually empty, poorly graded, off-beat, or like a template with text placed on top.

## Prompt To Give The Agent

```text
请使用仓库里的 cli-hub-matrix/video-creation/SKILL.md 作为主要工作指南，完成 docs/cli-matrix/test-plans/video-creation-agent-challenges.md 里的三个高难度视频任务。

要求：
- 开始前先安装并使用当前 worktree 里的本地最新版 cli-hub，确保能用最新的 matrix/preflight 能力。优先运行：`python3 -m pip install -e ./cli-hub`；如果当前环境不适合安装，就用 `PYTHONPATH=cli-hub python3 -m cli_hub.cli ...` 调用本地源码版本。
- 为本次测试创建独立输出目录，例如 `docs/cli-matrix/test-plans/video-challenge-runs/<run-id>/`，三个任务的 plan、sources、music_sources、qc、review frames 和 final output 都放进去。
- 先运行 video-creation matrix preflight，按任务目标、质量、可用工具和工作流选择 provider。
- 优先使用 `cli-hub-matrix/video-creation/SKILL.md` 里定义的 capabilities 和 providers，尤其是 `video.search`、`video.download`、`music.search`、`music.download`、`media.analyze`、`text.caption`、`composite.assemble`、`composite.overlay`、`package.encode`、`quality.review`。
- 每个任务开始剪辑前，必须按 `cli-hub-matrix/video-creation/references/story-structure-audio.md` 写 `creative_direction.md`：明确一句话承诺、故事弧线、情绪曲线、音频弧线、剪辑密度曲线、素材角色、转折点、高潮和结尾 payoff。不能只写素材列表或简单 beat map。
- 每个任务都必须使用真实 YouTube 和真实 Bilibili 视频作为主素材：至少各 1 个 YouTube 源和 1 个 Bilibili 源。`sources.json` 必须记录 title、platform、URL、download command、local file、使用片段时间段、用途。
- YouTube 下载可使用已经准备好的 cookie 文件：`docs/cli-matrix/test-plans/youtube-cookies.txt`。当 YouTube 需要登录状态、年龄验证、反爬限制或更高清格式时，用 `yt-dlp --cookies docs/cli-matrix/test-plans/youtube-cookies.txt ...`。不要把 cookie 文件内容写入日志、`sources.json`、报告或最终回答，只记录使用了该 cookie 文件路径。
- 大胆使用高质量互联网视频和音乐参考。这个测试只做本地学习和参考评估，不要因为素材来自 YouTube/Bilibili 就退回到低质量占位画面。
- 不要用 Pexels/Pixabay/stock footage 作为主素材；最多只能作为补充氛围镜头，并且必须在 `sources.json` 标为 secondary。
- 允许使用 skill/matrix 之外的工具或能力，但必须在对应任务的 `plan.md` 或 Notes 里标记为 “out-of-matrix”，并写清楚为什么需要它、它补足了哪个缺口、是否应该考虑回填到 matrix。
- out-of-matrix 不能成为低质捷径：不要用自写 Python/OpenCV/Pillow 几何图形、火柴人、简单字幕卡、静态 slide、复古矢量动画、随机粒子、低质合成 UI 来替代核心视频素材、高质量剪辑或高质量 motion graphics。
- 如果 YouTube 临时下载失败，优先切到 Bilibili 找同主题/同电影/同类型的高质量真实视频源；也可以切到其他高质量真实视频来源。`plan.md` 里必须说明切换原因、替代来源、质量是否足够，以及是否仍覆盖任务目标。不要用 stock 或自绘动画硬凑完成。
- 如果 Bilibili 临时下载失败，优先换 BV/合集/转载源、换 `yt-dlp`/`BBDown` 参数、换清晰度，或切到 YouTube/其他高质量真实视频来源。仍失败才把任务标记 blocked 并说明原因。
- 三个任务都要真实产出可播放视频，不要只写计划。
- 每完成一个任务，就在对应 checkbox 里打勾，并把最终视频路径填到 “Final output path”。
- 不要默认调用付费 API；如果确实需要，用 skill 里的 escalation 模板先向用户确认。
- 每个任务都必须做音乐/音频节奏设计：写 beat map 和 audio arc，按鼓点/重拍剪辑，至少包含若干处明确卡点、速度变化、冲击转场、声音停顿、音乐段落变化或 sound design hit。不能使用从头到尾不变的平铺音乐。
- 每个任务都必须有明显后期处理：调色/对比度、镜头裁切、节奏重排、转场、字幕/标题设计、音量混合。字幕/标题必须按 `cli-hub-matrix/video-creation/references/captions.md` 做成有设计感的 caption pass，不能是默认白字黑边 SRT。原素材顺序拼接不算完成。
- 做完每个视频后都跑 quality.review，至少检查时长、分辨率、音量/静音、黑帧、字幕可读性、首尾帧和中间帧。
- quality.review 还必须包含 creative review：保存 first/middle/last/contact-sheet 帧图，按 `story-structure-audio.md` 检查故事是否有起伏、内部逻辑、转折/高潮/payoff，音频是否有段落和变化，并写出 3 条“不够高级”的风险检查。如果视频看起来廉价、简陋、像模板、像素材粗拼、像上古动画、像随机片段合集、或者音乐平铺无聊，不能打勾，必须重做或标记 blocked。
- 每个任务做完后，必须启动一个单独的全新 context subagent 对该任务最终视频进行独立评估。不要把制作过程、预期答案或自评结论透露给评估 subagent；只给它最终视频路径、任务 brief、sources/qc 路径和“请严格判断是否达到高质量要求”。评估报告保存为 `<run-dir>/taskN/independent_review.md`。
- 只有该任务的独立 subagent 评估通过，才允许勾选 “Final output path”。如果评估不通过，必须根据评估意见重做或把任务标记 blocked。

请按顺序完成 Task 1、Task 2、Task 3。完成后给我三个最终视频路径、每个任务的来源摘要、剪辑策略和质量自评。
```

---

## Task 1 — 超燃体育音乐混剪

做一个 60-90 秒的高能体育音乐混剪，感觉像球星高光、赛事开场片、运动品牌广告或 B 站/YouTube 爆款燃向剪辑。你可以自选体育方向，比如篮球、足球、拳击、MMA、田径、滑板、赛车、极限运动，但必须聚焦一个明确主题。

必须使用真实 YouTube 和 Bilibili 运动视频作为主素材。不要用 stock 代替。画面要有高能动作、真实人物、冲突、速度、力量和情绪。音乐要强，剪辑要按重拍推进。

质量目标：看起来像一个认真做过的高质量 sports hype edit，而不是 stock footage montage。需要有高密度剪辑、速度 ramp、镜头冲击、节奏断点、标题设计、音效点缀和最终爆点。

- [x] Run preflight and write chosen providers to `<run-dir>/task1/plan.md`
- [x] Pick a concrete sports theme and write a 60-90s beat map
- [x] Search YouTube for high-quality sports source videos and download at least 1 usable source
- [x] Search Bilibili for high-quality sports source videos and download at least 1 usable source
- [x] Save all source URLs, titles, download commands, local files, and chosen timestamps to `<run-dir>/task1/sources.json`
- [x] Search/download or create a high-energy music bed; save details to `<run-dir>/task1/music_sources.json`
- [x] Select the strongest shots and cut to music beats, not source-video order
- [x] Add speed ramps, impact cuts, punch titles, sound hits, color/contrast treatment, and a strong ending
- [x] Run technical QC and creative review; save `<run-dir>/task1/qc_report.md` plus extracted review frames
- [x] Run a fresh-context independent subagent review and save `<run-dir>/task1/independent_review.md`
- [x] Final output path: `docs/cli-matrix/test-plans/video-challenge-runs/20260503T102127Z/task1/final_task1_nba_hype.mp4`

Notes:

---

## Task 2 — 真实电影剪辑缩短版解说

做一个 120-180 秒的真实电影剪辑缩短版解说。选择一部真实存在的电影，围绕电影里一个清晰段落、主线或高能情节，把它压缩成一个短视频解说：开头 5 秒强钩子，中间快速推进剧情和冲突，结尾给出反转、高潮或情绪落点。

这不是原创故事任务。不要自己编电影，不要找一些氛围素材硬配旁白。必须使用这部真实电影相关的 YouTube 和 Bilibili 视频作为主素材，例如电影片段、预告、解说素材、剪辑片段、官方/非官方片段等。画面要能看出来是在讲同一部电影。

质量目标：像一个成熟的电影解说短视频，而不是“旁白 + 黑暗 B-roll”。要有明确剧情顺序、镜头选择、人物/场景连续性、字幕节奏、音效/音乐烘托和压缩叙事。

- [x] Run preflight and write chosen providers to `<run-dir>/task2/plan.md`
- [x] Pick one real movie and write why this movie/segment works for a 120-180s commentary edit
- [x] Search YouTube for source videos about this exact movie and download at least 1 usable source
- [x] Search Bilibili for source videos about this exact movie and download at least 1 usable source
- [x] Save all source URLs, titles, download commands, local files, and chosen timestamps to `<run-dir>/task2/sources.json`
- [x] Write a concise Chinese commentary script with hook, setup, conflict, escalation, and ending
- [x] Generate or record narration; create subtitles from the narration/script
- [x] Add music/SFX and mix levels so narration is clear but the edit still feels cinematic
- [x] Assemble the film commentary with scene continuity, tight pacing, captions, title cards, and a satisfying ending
- [x] Run technical QC and creative review; save `<run-dir>/task2/qc_report.md` plus extracted review frames
- [x] Run a fresh-context independent subagent review and save `<run-dir>/task2/independent_review.md`
- [x] Final output path: `docs/cli-matrix/test-plans/video-challenge-runs/20260503T102127Z/task2/final_task2_big_buck_bunny_commentary.mp4`

Notes:

---

## Task 3 — 游戏/机甲/科幻 CG 高燃预告混剪

做一个 60-90 秒的游戏、机甲、科幻或赛博 CG 高燃预告混剪。可以围绕一个主题，比如“机甲降临”“赛博都市反击”“游戏角色觉醒”“末日舰队”“二次元/游戏 CG 燃向混剪”。目标是做成一个像新游预告、动画 PV 或 B 站燃向 GMV/AMV 的短片。

必须使用真实 YouTube 和 Bilibili 视频作为主素材。优先找高质量 game cinematic、CG trailer、anime PV、Bilibili 燃向混剪、官方预告、游戏角色演示等。不要用自绘 UI、随机粒子、几何框、静态 feature card 来交差。

质量目标：高视觉密度、高冲击剪辑、高质量音乐卡点。要有开场钩子、铺垫、第一次爆点、中段节奏变化、最终高潮和片尾标题。至少做出几个明显的转场/遮罩/闪白/震动/速度变化/音效 hit。

- [x] Run preflight and write chosen providers to `<run-dir>/task3/plan.md`
- [x] Pick a concrete theme and write a 60-90s trailer-style beat map
- [x] Search YouTube for high-quality game/anime/CG source videos and download at least 1 usable source
- [x] Search Bilibili for high-quality game/anime/CG source videos and download at least 1 usable source
- [x] Save all source URLs, titles, download commands, local files, and chosen timestamps to `<run-dir>/task3/sources.json`
- [x] Search/download a powerful music bed or use a strong source soundtrack; save details to `<run-dir>/task3/music_sources.json`
- [x] Cut shots to the music with clear buildup, drop, mid-section variation, and final climax
- [x] Add high-impact typography, transitions, color/contrast treatment, SFX hits, and a final title lockup
- [x] Run technical QC and creative review; save `<run-dir>/task3/qc_report.md` plus extracted review frames
- [x] Run a fresh-context independent subagent review and save `<run-dir>/task3/independent_review.md`
- [x] Final output path: `docs/cli-matrix/test-plans/video-challenge-runs/20260503T102127Z/task3/final_task3_tears_of_steel_trailer.mp4`

Notes:
