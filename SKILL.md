---
name: novel-to-short-video
description: Convert novel text into a loopable short-form video by extracting the single most compelling story segment, generating images, and assembling a Remotion render. Use when users ask to turn novels or chapters into a 50-60 second narrative short, teaser reel, or looping story clip that feels self-contained yet leaves strong lingering curiosity.
---

# Novel to Short Video

## Core Dependencies

Always coordinate these skills in order:

1. `openai-text-to-image-storyboard` for storyboard image generation.
2. `remotion-best-practices` for composition and rendering in Remotion.

If a user uses approximate names (for example "openai-text-to-images" or "remotionbest-practises"), normalize to the exact skill names above.

## Required Inputs

Collect the minimum required inputs before execution:

- `project_dir` (absolute path)
- `content_name` (output folder/video name)
- novel content (raw text or file path)
- output orientation/resolution (default: vertical 1080x1920)

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
  - `story_key_line`

### 2) Generate a pre-production plan markdown (required)

- Before image/video generation, create:
  - `<project_dir>/docs/plans/`
  - `<project_dir>/docs/plans/<YYYY-MM-DD>-<chapter_slug>.md`
- `chapter_slug` should come from the chapter name/number and be filesystem-safe.
- Build the plan content from template file:
  - `references/plan-template.md`
- The plan markdown must include:
  - selected highlight segment details,
  - story script and beat-level timing for the full video,
  - standalone-story check and lingering-question design,
  - image assets that will be generated for the segment.
- All template guidance/placeholders must be wrapped in square brackets (for example `[fill_this]`).
- After filling content, remove every placeholder/guidance marker from the final plan file.
- Enforce 1:1 mapping: one selected highlight segment must map to one full 50-60 second video.

### 3) Request user approval before execution (required)

- Present the generated plan markdown to the user (path + concise summary).
- Ask for explicit approval.
- Do not start image/render execution until user explicitly agrees.
- If user asks for edits, update the plan and request approval again.

### 4) Build a loopable 50-60s story script

- Produce one short video with total duration in **50-60 seconds**.
- Build pacing within the same segment using beat progression (hook -> escalation -> climax -> loop closure).
- Make the segment self-contained as a mini-story:
  - viewers can understand setup, conflict, turning point, and immediate outcome without prior chapter context,
  - the story script provides enough context in-line rather than relying on outside exposition.
- Write the story script so:
  - first sentence is the hook,
  - final sentence closes the loop by reusing or tightly paraphrasing the first sentence,
  - ending still leaves one unresolved high-stakes question so viewers feel "意猶未盡".
- Ensure the final visual beat can cut/fade back to the opening frame naturally.
- Use the plan markdown as the source of truth and keep beat order aligned with it.

If producing multiple short videos in one request, enforce the same 50-60 second duration for each output.

### 5) Generate storyboard images (`openai-text-to-image-storyboard`)

- Create `prompts.json` under:
  - `<project_dir>/pictures/<content_name>/prompts.json`
- Provide beat-based prompts in narrative order, reusing recurring character skeletons for consistency.
- Ensure prompts match the image list defined in the plan markdown.
- Generate images with:

```bash
python /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/scripts/generate_storyboard_images.py \
  --project-dir "<project_dir>" \
  --env-file /Users/tszkinlai/.codex/skills/openai-text-to-image-storyboard/.env \
  --content-name "<content_name>" \
  --prompts-file "<project_dir>/pictures/<content_name>/prompts.json"
```

### 6) Compose and render (`remotion-best-practices`)

- Build or reuse Remotion workspace:
  - `<project_dir>/video/<content_name>/remotion/`
- Load Remotion rules as needed (at minimum):
  - `rules/compositions.md`
  - `rules/transitions.md`
- Implement beat sequencing with clean visual pacing based on the plan script.
- Keep one contiguous narrative segment so the plan's segment-to-video mapping remains valid.
- Add loop closure in the tail section:
  - final 1-2 seconds visually connect back to opening shot,
  - final script line closes back to opening hook.
- Render MP4 output to:
  - `<project_dir>/video/<content_name>/renders/`

### 7) Keep Remotion project and enforce `.gitignore`

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
- generated image directory path
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
- user explicitly approved the plan before image/render steps
- one selected segment maps to one full output video
- duration is within 50-60 seconds per output video
- opening and ending lines form a narrative loop
- ending leaves one unresolved but compelling question
- image assets are generated successfully
- Remotion project is preserved and `.gitignore` is configured
