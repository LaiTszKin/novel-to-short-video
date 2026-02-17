---
name: novel-to-short-video
description: Convert novel text into a loopable short-form video by extracting the single most compelling story segment, generating images and voiceover, and assembling a Remotion render. Use when users ask to turn novels or chapters into a 50-60 second narrative short, teaser reel, or looping story clip that feels self-contained yet leaves strong lingering curiosity.
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

### 1) Extract one highlight segment

- Parse the novel and list candidate high-tension segments.
- Score each segment by conflict intensity, stakes, emotional swing, turning-point value, visual concreteness, standalone readability, and cliffhanger potential.
- Select exactly **1** segment with the strongest overall dramatic impact.
- Reject segments that cannot be understood without heavy external context from other chapters.
- Prefer segments that can deliver a complete mini-arc while still leaving one meaningful unresolved question.
- Keep a structured segment sheet:
  - `title`
  - `source_excerpt`
  - `core_conflict`
  - `tension_arc` (setup -> escalation -> climax -> aftershock)
  - `standalone_story_basis`
  - `lingering_question`
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
  - selected highlight segment details,
  - narration script and beat-level timing for the full video,
  - standalone-story check and lingering-question design,
  - image assets that will be generated for the segment.
- All template guidance/placeholders must be wrapped in square brackets (for example `[fill_this]`).
- After filling content, remove every placeholder/guidance marker from the final plan file.
- Enforce 1:1 mapping: one selected highlight segment must map to one full 50-60 second video.

### 3) Request user approval before execution (required)

- Present the generated plan markdown to the user (path + concise summary).
- Ask for explicit approval.
- Do not start image/voice/render execution until user explicitly agrees.
- If user asks for edits, update the plan and request approval again.

### 4) Build a loopable 50-60s script

- Produce one short video with total duration in **50-60 seconds**.
- Build pacing within the same segment using beat progression (hook -> escalation -> climax -> loop closure).
- Make the segment self-contained as a mini-story:
  - viewers can understand setup, conflict, turning point, and immediate outcome without prior chapter context,
  - the narration provides enough context in-line rather than relying on outside exposition.
- Write narration so:
  - first sentence is the hook,
  - final sentence closes the loop by reusing or tightly paraphrasing the first sentence,
  - ending still leaves one unresolved high-stakes question so viewers feel "意猶未盡".
- Ensure the final visual beat can cut/fade back to the opening frame naturally.
- Use the plan markdown as the source of truth and keep beat order aligned with it.

If producing multiple short videos in one request, enforce the same 50-60 second duration for each output.

### 5) Create `roles.json` for recurring character prompts (required)

- Before creating/updating `roles.json`, read:
  - `references/roles-json.md`
- Create `roles.json` under:
  - `<project_dir>/pictures/<content_name>/roles.json`
- If `roles.json` does not exist, create it first before any `prompts.json` planning or generation.
- If `roles.json` already exists, read existing role prompts first and reuse matching roles.
- If a required role is missing, append a new role prompt entry for that role (do not overwrite existing entries).
- Save recurring character prompt skeletons using the schema in `references/roles-json.md`.
- If the selected segment has no recurring roles, still create `roles.json` with `{"characters": []}`.
- Keep role IDs stable and reuse them in every scene prompt.

### 6) Generate storyboard images (`openai-text-to-image-storyboard`)

- Create `prompts.json` under:
  - `<project_dir>/pictures/<content_name>/prompts.json`
- Use the structured JSON prompt format from `openai-text-to-image-storyboard`.
- Copy top-level `characters` from the final `roles.json` (reused + newly appended roles), then build beat-aligned `scenes` in narrative order.
- Ensure prompts match the image list defined in the plan markdown.
- Generate images with:

```bash
python /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/scripts/generate_storyboard_images.py \
  --project-dir "<project_dir>" \
  --env-file /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/.env \
  --content-name "<content_name>" \
  --prompts-file "<project_dir>/pictures/<content_name>/prompts.json"
```

### 7) Generate narration and subtitles (`docs-to-voice`)

- Use the loop script text as narration input.
- Generate audio + timeline + SRT:

```bash
python /Users/tszkinlai/.codex/skills/docs-to-voice/scripts/docs_to_voice.py \
  --project-dir "<project_dir>" \
  --project-name "<content_name>" \
  --text "<loop_narration_script>"
```

### 8) Compose and render (`remotion-best-practices`)

- Build or reuse Remotion workspace:
  - `<project_dir>/video/<content_name>/remotion/`
- Load Remotion rules as needed (at minimum):
  - `rules/compositions.md`
  - `rules/audio.md`
  - `rules/subtitles.md`
  - `rules/transitions.md`
- Implement beat sequencing with subtitle sync from SRT.
- Keep one contiguous narrative segment so the plan's segment-to-video mapping remains valid.
- Add loop closure in the tail section:
  - final 1-2 seconds visually connect back to opening shot,
  - final spoken line closes back to opening hook.
- Render MP4 output to:
  - `<project_dir>/video/<content_name>/renders/`

### 9) Keep Remotion project and enforce `.gitignore`

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

- selected highlight-segment summary (with why it was selected)
- proof note that the segment is self-contained and why the ending still leaves viewers wanting more
- plan markdown path (`<project_dir>/docs/plans/<YYYY-MM-DD>-<chapter_slug>.md`)
- explicit user approval confirmation (before asset generation)
- roles prompt file path (`<project_dir>/pictures/<content_name>/roles.json`)
- prompts file path (`<project_dir>/pictures/<content_name>/prompts.json`)
- generated image directory path
- narration audio path
- subtitle SRT path
- final rendered video path
- Remotion project path (retained for adjustments)

## Quality Gate Checklist

Before finishing, verify all conditions:

- exactly 1 highest-impact highlight segment selected
- selected segment is understandable as a standalone mini-story
- plan markdown exists in `docs/plans/` with date + chapter naming
- plan content starts from `references/plan-template.md`
- plan markdown includes the selected segment, beat/script details, standalone-story check, lingering-question design, and segment image generation list
- all bracketed placeholders/guidance are removed from the final filled plan
- user explicitly approved the plan before image/voice/render steps
- `roles.json` exists and follows `references/roles-json.md`
- existing role prompts are reused when available; new role prompts are added only when missing
- `prompts.json` uses structured mode and reuses role IDs from `roles.json`
- one selected segment maps to one full output video
- duration is within 50-60 seconds per output video
- opening and ending lines form a narrative loop
- ending leaves one unresolved but compelling question
- images and voice assets are generated successfully
- Remotion project is preserved and `.gitignore` is configured
