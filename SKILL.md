---
name: novel-to-short-video
description: Convert novel text into loopable short-form videos by extracting the 3 highest-tension scenes, generating images and voiceover, and assembling a Remotion render. Use when users ask to turn novels or chapters into 50-60 second narrative shorts, teaser reels, or looping story clips.
---

# Novel to Short Video

## Core Dependencies

Always coordinate these skills in order:

1. `openai-text-to-image-storyboard` for storyboard image generation.
2. `docs-to-voice` for narration audio, sentence timeline, and SRT.
3. `remotion-best-practices` for composition and rendering in Remotion.

If a user uses approximate names (for example "openai-text-to-images" or "remotionbest-practises"), normalize to the exact skill names above.

## Required Inputs

Collect the minimum required inputs before execution:

- `project_dir` (absolute path)
- `content_name` (output folder/video name)
- novel content (raw text or file path)
- output orientation/resolution (default: vertical 1080x1920)
- narration language/voice preferences when provided

If critical inputs are missing, ask concise follow-up questions.

## Workflow

### 1) Extract 3 high-tension scenes

- Parse the novel and list candidate scenes.
- Score each scene by conflict intensity, stakes, emotional swing, turning-point value, and visual concreteness.
- Select exactly **3** scenes with the strongest tension while preserving narrative coherence.
- Keep a structured scene sheet for each selected scene:
  - `title`
  - `source_excerpt`
  - `core_conflict`
  - `visual_beats`
  - `narration_key_line`

### 2) Generate a pre-production plan markdown (required)

- Before image/voice/video generation, create:
  - `<project_dir>/docs/plans/`
  - `<project_dir>/docs/plans/<YYYY-MM-DD>-<chapter_slug>.md`
- `chapter_slug` should come from the chapter name/number and be filesystem-safe.
- Build the plan content from template file:
  - `references/plan-template.md`
- The plan markdown must include:
  - scene list,
  - script text for each scene,
  - image assets that will be generated for each scene.
- All template guidance/placeholders must be wrapped in square brackets (for example `[fill_this]`).
- After filling content, remove every placeholder/guidance marker from the final plan file.
- Enforce 1:1 mapping: each selected scene must support one video segment.

### 3) Request user approval before execution (required)

- Present the generated plan markdown to the user (path + concise summary).
- Ask for explicit approval.
- Do not start image/voice/render execution until user explicitly agrees.
- If user asks for edits, update the plan and request approval again.

### 4) Build a loopable 50-60s script

- Produce one short video with total duration in **50-60 seconds**.
- Distribute timing across the 3 scenes (roughly even pacing plus transitions).
- Write narration so:
  - first sentence is the hook,
  - final sentence closes the loop by reusing or tightly paraphrasing the first sentence.
- Ensure the final visual beat can cut/fade back to the opening frame naturally.
- Use the plan markdown as the source of truth and keep scene order aligned with it.

If producing multiple short videos in one request, enforce the same 50-60 second duration for each output.

### 5) Generate storyboard images (`openai-text-to-image-storyboard`)

- Create `prompts.json` under:
  - `<project_dir>/pictures/<content_name>/prompts.json`
- Provide scene prompts in narrative order, reusing recurring character skeletons for consistency.
- Ensure prompts match the images listed in the plan markdown.
- Generate images with:

```bash
python /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/scripts/generate_storyboard_images.py \
  --project-dir "<project_dir>" \
  --env-file /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/.env \
  --content-name "<content_name>" \
  --prompts-file "<project_dir>/pictures/<content_name>/prompts.json"
```

### 6) Generate narration and subtitles (`docs-to-voice`)

- Use the loop script text as narration input.
- Generate audio + timeline + SRT:

```bash
python /Users/tszkinlai/.codex/skills/docs-to-voice/scripts/docs_to_voice.py \
  --project-dir "<project_dir>" \
  --project-name "<content_name>" \
  --text "<loop_narration_script>"
```

### 7) Compose and render (`remotion-best-practices`)

- Build or reuse Remotion workspace:
  - `<project_dir>/video/<content_name>/remotion/`
- Load Remotion rules as needed (at minimum):
  - `rules/compositions.md`
  - `rules/audio.md`
  - `rules/subtitles.md`
  - `rules/transitions.md`
- Implement scene sequencing with subtitle sync from SRT.
- Keep one contiguous segment per scene so the plan's scene-to-segment mapping remains valid.
- Add loop closure in the tail section:
  - final 1-2 seconds visually connect back to opening shot,
  - final spoken line closes back to opening hook.
- Render MP4 output to:
  - `<project_dir>/video/<content_name>/renders/`

### 8) Keep Remotion project and enforce `.gitignore`

Preserve Remotion project sources for user revisions. Do not delete project files after rendering.

Ensure `<project_dir>/video/<content_name>/remotion/.gitignore` includes at least:

```gitignore
node_modules/
out/
dist/
.cache/
*.log
.DS_Store
```

## Output Contract

Return:

- selected 3-scene summary (with why each was selected)
- plan markdown path (`<project_dir>/docs/plans/<YYYY-MM-DD>-<chapter_slug>.md`)
- explicit user approval confirmation (before asset generation)
- generated image directory path
- narration audio path
- subtitle SRT path
- final rendered video path
- Remotion project path (retained for adjustments)

## Quality Gate Checklist

Before finishing, verify all conditions:

- exactly 3 high-tension scenes selected
- plan markdown exists in `docs/plans/` with date + chapter naming
- plan content starts from `references/plan-template.md`
- plan markdown includes scenes, per-scene scripts, and per-scene image generation list
- all bracketed placeholders/guidance are removed from the final filled plan
- user explicitly approved the plan before image/voice/render steps
- each scene maps to one video segment
- duration is within 50-60 seconds per output video
- opening and ending lines form a narrative loop
- images and voice assets are generated successfully
- Remotion project is preserved and `.gitignore` is configured
