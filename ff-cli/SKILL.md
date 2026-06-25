---
name: ff-cli
description: Work with FlutterFlow AI CLI workspaces and existing FlutterFlow projects. Use when you need to inspect FlutterFlow pages/components, edit dsl/edit.dart, use generated_code as read-only runtime truth, wire native FlutterFlow actions/API calls/Supabase queries, validate and push with flutterflow ai, or avoid non-editable generated Dart/custom-widget changes.
---

# FlutterFlow AI CLI

Version: 0.2.3
Updated: 2026-06-25

Change log:
- 0.2.3 - Added a battle-tested pointer for custom-code artifact types (class/enum vs widget storage; widgets are not a UI-only edit).
- 0.2.2 - Added a battle-tested pointer for version-history diffing (refresh-context loaded-version scope; non-destructive restore).
- 0.2.1 - Added a battle-tested pointer for action-chain and grouped local-state update wiring.
- 0.2.0 - Added repeatable sync guidance for keeping default `dsl/edit.dart` limited to custom-code sync, sectioning it by type, and asking before retaining one-off UI/page/component migrations after a successful push.
- 0.1.0 - Initial public release with core workflow rules and pointers to battle-tested lessons.

## Core Rules

- Treat `generated_code/` as read-only runtime truth. Inspect it to understand actual Flutter output, but change FlutterFlow through `dsl/edit.dart` or FlutterFlow UI steps.
- For existing projects, edit `dsl/edit.dart`, not `dsl/create.dart`.
- Prefer FlutterFlow-native resources over custom Dart: API Calls, Supabase queries, action blocks, built-in formatters, app/page state, component params, and existing functions/actions.
- Do not create custom actions/functions/widgets when a FlutterFlow-native API/action/binding can do the job and remain editable in the UI.
- When custom code is unavoidable, keep it narrow and non-visual unless the user explicitly approves a custom widget.
- Do not patch schema dumps. Use dumps only as reference and provide standalone SQL files for proposed DB changes.
- For existing Supabase projects, read `references/battle-tested-lessons.md` before using `app.supabase`, `app.table`, `Postgres*` actions, or action-block wrappers for Supabase mutations.
- For action-chain cleanup, many-field page/component state assignments, or raw local-state update mutations, read the Action Wiring section in `references/battle-tested-lessons.md`.

## Edit Workflow

1. Read `PROJECT_CONTEXT.md`, `context/api_calls.md`, and relevant generated Dart for the page/component.
2. Inspect before editing:
   ```bash
   flutterflow ai inspect <project-id> --page <PageName> --dsl-json
   ```
   Use `--component` for components. If the user provides a selector path, inspect that selector first.
3. Make changes in `dsl/edit.dart`.
4. Validate locally:
   ```bash
   dart test
   flutterflow ai validate dsl/edit.dart --project-id <project-id>
   ```
5. Push only after validation:
   ```bash
   flutterflow ai run dsl/edit.dart --project-id <project-id> --commit-message "<short change summary>"
   flutterflow ai refresh-context <project-id>
   ```
6. After refresh, inspect `generated_code/` to confirm FlutterFlow generated what was intended.

## Repeatable Sync And One-Off Migrations

- Keep the default `ai-workspace/dsl/edit.dart` limited to safe repeatable sync work, especially repo-owned custom code, custom action metadata, custom functions, and Code Files/classes.
- Section the default `dsl/edit.dart` by sync type so stale mutations are easy to spot. Prefer a shape like `syncCustomFunctions(project)`, `syncCodeFiles(project)`, `syncCustomActions(project)`, `syncCustomActionMetadata(project)`, then shared helpers.
- After a successful UI/page/component migration push, ask the user whether to clear or move those one-off mutations out of the default `dsl/edit.dart`. Explain that if they are left in place, a later unrelated push can replay them and overwrite live FlutterFlow server changes, FlutterFlow UI edits, or another thread's page/component updates.
- Do not leave page/component/layout/action-chain migrations in the default `dsl/edit.dart` after they have been pushed successfully unless the user explicitly wants that migration to remain repeatable.
- Put one-off FlutterFlow UI/page/component migrations in separate dated files under `ai-workspace/dsl/migrations/`, and run them explicitly only for that task.
- Before any unrelated custom-code push, check `dsl/edit.dart` for stale `editPage`, `editComponent`, raw `findPage(...)`/`findComponent(...)` mutation blocks, page state migrations, layout changes, action-chain rewiring, or old node-key repairs. Remove, move, or gate them unless the current task is intentionally changing that UI.
- `flutterflow ai refresh-context` updates context/generated snapshots, but it does not update or clean `dsl/edit.dart`.
- When another thread or the FlutterFlow UI changes a page/component, do not assume refreshing this workspace makes `dsl/edit.dart` safe. Inspect and prune stale page/component mutations before pushing.

## API Calls

Before adding custom RPC/HTTP code, inspect FlutterFlow API Calls:

```bash
sed -n '1,220p' context/api_calls.md
rg -n "GroupName|CallName|endpoint_name" generated_code/lib/backend/api_requests/api_calls.dart
```

Use FlutterFlow's API Calls library as the editable integration surface:

- If the needed REST/RPC/Edge call already exists, wire that API Call in FlutterFlow actions.
- If the needed call does not exist, create it as a FlutterFlow API Call instead of writing custom HTTP/RPC Dart.
- If an API group already exists with the same base URL, add the new call to that group.
- Create a new API group only when no existing group has the correct base URL or auth/header pattern.
- Keep headers, auth token usage, request body, response parsing, and test values editable in FlutterFlow's API Calls UI.

## Visual And Runtime Bugs

- For overflow, layout, null crashes, unbounded constraints, bad generated imports, or "looks wrong," read the generated Dart for the page after finding the entity in `generated_code/.flutterflow/export_manifest.json`.
- A passing `validate` does not prove generated Dart is good. Run targeted analysis when custom code or risky generated bindings are involved:
  ```bash
  dart analyze lib/custom_code/actions/<file>.dart
  dart analyze lib/<page>/<page>_widget.dart lib/<page>/<page>_model.dart
  ```
- FlutterFlow-generated warnings are common; distinguish warnings from compile errors.

## App Raw Caution

- Use `app.raw()` only when the typed DSL cannot express the change.
- Verify raw changes with `flutterflow ai refresh-context` and generated code inspection because some raw mutations can compile but not persist as expected.
- For custom action calls in raw proto wiring, prefer the existing project's generated/action shape. Older custom actions use `FFAction.customAction` with `FFCustomActionCall`; newer inline custom code uses `customCodeCall`.

## More Details

Read `references/battle-tested-lessons.md` when working with existing Supabase projects, action wiring, many-field state updates, custom code upserts, custom-code artifact types (class vs widget storage), raw proto action wiring, component limitations, schema corruption recovery, version-history diffing, or other fragile FlutterFlow AI CLI behavior.
