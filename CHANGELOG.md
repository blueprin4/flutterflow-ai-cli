# Changelog

Version: 0.2.3
Updated: 2026-06-25

## 0.2.3 - 2026-06-25

- Added a "Custom Code Artifact Types And Storage" note: custom classes/enums attach to an `FFCustomCodeFile` and push cleanly via `updateCustomClass`/`updateCustomEnum`, while custom widgets store code on `FFWidgetClass`.
- Clarified that custom widgets are still pushed via the SDK (`addCustomWidget`/`updateCustomWidget`), not a UI-only edit, with the UI editor as a fallback only if an update silently does not take.

## 0.2.2 - 2026-06-25

- Added a "Version History And Diffing" note: `flutterflow ai refresh-context` captures only the currently-loaded project version, so diffing two commits requires restore + refresh on each.
- Documented that restoring a FlutterFlow version is non-destructive (newer commits stay in history), enabling free bisecting of regressions.

## 0.2.1 - 2026-06-02

- Added action-wiring guidance for grouping many page/component state assignments into one FlutterFlow local-state update action.
- Added guidance to preserve designer-built layouts and non-state follow-up actions when cleaning up existing action chains.
- Added generated Dart verification expectations after grouping local-state updates.

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
