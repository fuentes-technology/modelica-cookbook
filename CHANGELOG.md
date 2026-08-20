# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-08-20

### Added

- `author-modelica-model-mcp` skill: the OMEdit-MCP-server counterpart to
  `author-modelica-model` — authors and validates a Modelica model through a live
  OMEdit MCP session (check + simulate) instead of the `omc` CLI, and can save the
  result to a `.mo` file on request.

### Changed

- Common steps shared by both `author-modelica-model*` skills (resolving what to
  author, model authoring style, the check/simulate checkpoint loop and its pass
  criteria, and the report checklist) are now factored into `skills/shared/` instead
  of duplicated per skill.

## [0.1.0] - 2026-08-11

### Added

- `modelica-skills` plugin and `modelica-cookbook` marketplace registration.
- `author-modelica-model` skill: writes a Modelica `.mo` model and validates it with
  OpenModelica's `checkModel` and `simulate` checkpoints, fixing and re-running until
  both pass or reporting why they don't.

[Unreleased]: https://github.com/fuentes-technology/modelica-cookbook/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/fuentes-technology/modelica-cookbook/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/fuentes-technology/modelica-cookbook/releases/tag/v0.1.0
