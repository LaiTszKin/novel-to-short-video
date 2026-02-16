# Changelog

All notable changes to this project will be documented in this file.

## [0.1.0] - 2026-02-16

### Added
- Require a pre-production plan markdown before generation, including scenes, per-scene scripts, and image lists.
- Add `references/plan-template.md` as the canonical plan template with bracketed placeholders.
- Add workflow gate requiring explicit user approval of the plan before image, voice, and render steps.

### Changed
- Update skill workflow numbering and quality checks to enforce plan-first execution.
- Update README and default agent prompt to match the new plan approval flow.
