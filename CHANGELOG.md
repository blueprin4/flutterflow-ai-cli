# Changelog

Version: 0.2.0
Updated: 2026-05-27

## 0.2.0 - 2026-05-27

- Added repeatable sync guidance for keeping the default `dsl/edit.dart` limited to custom-code sync.
- Added guidance to section `dsl/edit.dart` by custom functions, Code Files/classes, custom actions, metadata, and shared helpers.
- Added a post-push prompt rule: after successful UI/page/component migration pushes, ask whether to clear or move one-off mutations out of the default `dsl/edit.dart` and warn that leaving them can overwrite live FlutterFlow server/UI changes on later pushes.

## 0.1.1 - 2026-05-11

- Clarified that native-first guidance is partly a maintainability preference and partly a workaround for observed CLI failure modes.
- Clarified that direct component editing can work; the weaker pattern is trying to edit component internals indirectly from a page.
- Softened the schema update warning into a diagnostic caution for unexpected metadata or validation regressions.

## 0.1.0 - 2026-05-11

- Added the public `ff-cli` Codex skill.
- Added sanitized battle-tested lessons for existing FlutterFlow projects, Supabase metadata, generated-code verification, and action-block parameter wiring.
