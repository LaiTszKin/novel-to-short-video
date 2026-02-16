# Changelog

All notable changes to this project will be documented in this file.

## [0.2.1] - 2026-02-16

### Changed
- Strengthen selection criteria to require a standalone mini-story arc plus lingering-question potential.
- Update plan template with standalone-story basis, lingering question, and story satisfaction checks.
- Update README and default agent prompt to explicitly enforce "完整但意猶未盡" narrative outcomes.

## [0.2.0] - 2026-02-16

### Changed
- Replace 3-scene extraction with single highlight-segment extraction for stronger short-video focus.
- Update planning template to a single-segment structure with beat-level timing and asset planning.
- Align skill workflow, quality gate checklist, README, and agent default prompt with the 50-60 second single-segment pipeline.

## [0.1.0] - 2026-02-16

### Added
- Require a pre-production plan markdown before generation, including scenes, per-scene scripts, and image lists.
- Add `references/plan-template.md` as the canonical plan template with bracketed placeholders.
- Add workflow gate requiring explicit user approval of the plan before image, voice, and render steps.

### Changed
- Update skill workflow numbering and quality checks to enforce plan-first execution.
- Update README and default agent prompt to match the new plan approval flow.
