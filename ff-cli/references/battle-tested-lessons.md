# Battle-Tested FlutterFlow AI CLI Lessons

Version: 0.1.0  
Updated: 2026-05-11

Change log:
- 0.1.0 - Initial public release with sanitized lessons for existing Supabase projects, action-block arguments, and generated-code verification.

## Existing Supabase Projects

An existing FlutterFlow project can already generate valid Supabase table classes while the AI DSL still needs careful backend declarations.

- Detect existing Supabase through generated code first: `generated_code/lib/backend/supabase/supabase.dart` and `generated_code/lib/backend/supabase/database/tables/`.
- Do not use bare `app.supabase(url: ..., anonKey: ...)` in an existing OAuth-connected Supabase project. It can disturb FlutterFlow's project-side Supabase metadata and produce project-wide validation errors like `Table not set`, `table not found`, and `Invalid postgres row field operation`.
- If `app.table(...)` is needed, declare Supabase with connected metadata before the table:
  ```dart
  app.supabase(
    url: '<supabase-url>',
    anonKey: '<supabase-anon-key>',
    connectedProjectId: '<supabase-project-ref>',
    connectedProjectName: '<supabase-project-name>',
  );
  ```
  Add `connectedRegion` too if the project exposes it.
- `app.table(...)` alone does not reuse FlutterFlow's existing table catalog; the DSL compiler requires `app.supabase(...)` or `app.postgres(...)` first.
- Validate in tiny increments: connected `app.supabase`, then `app.table`, then typed page/component state or params, then display bindings, then actions.
- Typed table state, component params, and row display bindings can validate with connected `app.supabase(...)` plus `app.table(...)`.
- Direct `PostgresQuery` / `PostgresUpdate` actions inside existing widget edits such as `editPageOnLoad(...)` or `editComponent(...ensureActions...)` can fail with `requires app.supabase(...) or app.postgres(...) to be configured first`. Current edit-action compiler paths may not see the outer SQL backend declaration.
- Workaround for existing button mutations: put the SQL mutation in a top-level `app.actionBlock(...)`, then wire the existing widget to that block. For already-created blocks, use `ActionBlock.named('blockName', scope: ActionBlockLookupScope.app)` in `ExecuteActionBlock(...)`; do not pass a freshly declared `ActionBlockHandle` from the same edit when patching an existing widget.
- Action-block argument bug pattern: `ExecuteActionBlock(reviewAction, params: ...)` can validate but store argument values under newly generated parameter IDs. Generated Dart may then call `action_blocks.reviewAction(context)` without named arguments. The project has argument values, but their keys do not match the real action block parameter identifiers. Fix by referencing the existing block with `ActionBlock.named(...)` so argument keys match the existing action block params.
- If a prior run already wrote bad `FFExecuteActionComponent.argumentValues`, `ensureActions(...)` may consider the fixed action semantically equal and skip replacement. Force replacement by clearing the target trigger in an `app.raw(...)` mutation before `editComponent(...ensureActions...)`, then re-add the action with `ActionBlock.named(...)`:
  ```dart
  app.raw((project) {
    final component = findComponent(project, name: 'approvalCardComp');
    if (component == null) {
      return;
    }
    for (final buttonKey in ['Button_approve', 'Button_deny']) {
      final button = findByKey(component.node, buttonKey);
      button?.triggerActions.removeWhere(
        (trigger) => trigger.trigger.triggerType == FFActionTriggerType.ON_TAP,
      );
    }
  });

  app.editComponent('approvalCardComp', (component) {
    final review = ActionBlock.named(
      'reviewRecord',
      scope: ActionBlockLookupScope.app,
    );
    component.ensureActions(
      component.findByKey('Button_approve'),
      triggerType: FFActionTriggerType.ON_TAP,
      actions: [
        ExecuteActionBlock(
          review,
          params: {
            'recordRow': Param('recordRow'),
            'approvalStatus': 'approved',
            'active': true,
          },
        ),
        Snackbar('Approved'),
      ],
    );
  });
  ```
- After pushing an action-block call fix, run `flutterflow ai refresh-context <project-id>` and inspect generated Dart. The button should call the block with named args, for example `recordRow: widget.recordRow`.
- Query flows that must populate page-local state remain fragile through CLI. Prefer FlutterFlow UI, an existing API/RPC call, or isolate in a throwaway workspace before pushing.

## DSL Capabilities

Good fit:
- Creating pages, components, structs, enums, custom actions/functions/widgets.
- Adding pub dependencies.
- Setting page state fields.
- Simple widget edits such as text, padding, color, visibility, and bindings.
- Existing Supabase row display bindings when table metadata is declared with connected Supabase metadata.
- Removing entities.

Weak fit:
- Editing actions inside existing components from a page.
- Modifying existing trigger chains without replacing them.
- Direct SQL query/update actions inside existing page/component trigger edits.
- Complex custom action parameter mapping unless using raw proto carefully.

## Native First

Prefer FlutterFlow-native surfaces before custom code:
- API Calls for REST/RPC functions.
- Supabase queries/mutations where supported.
- Built-in formatters for currency/date/number display.
- Action blocks for reusable visible workflows.
- App/page state and component params for editor-visible state.

Use custom code only for gaps FlutterFlow cannot represent.

## Custom Code Upserts

`app.customAction()` throws on duplicate. In existing projects, use helper APIs inside `app.raw()`:

```dart
final existing = findCustomAction(project, name: 'myAction');
if (existing == null) {
  addCustomAction(project, name: 'myAction', code: code, description: '...');
} else {
  updateCustomAction(project, name: 'myAction', code: code, description: '...');
}
```

Same idea applies to `findCustomFunction` / `addCustomFunction` / `updateCustomFunction` and widgets.

## Raw Action Wiring

- `app.raw()` mutations run in registration order. Upsert actions/functions before raw wiring references them.
- Raw custom action calls may need `FFAction.outputVariableName`.
- Raw custom action args use `FFFunctionCallValues` keyed by parameter identifier key, not visible name.
- Page/local state variables in raw action arguments often need `nodeKeyRef`.
- Remove the existing trigger before replacing it so reruns stay idempotent.

## API Calls

Look here before writing custom RPC/client code:

```bash
sed -n '1,220p' context/api_calls.md
sed -n '1,140p' generated_code/lib/backend/api_requests/api_calls.dart
```

Generated API call code shows the group class, call class, params, headers, auth token, body, and response accessors.

## Generated Code

- Never edit `generated_code/` directly.
- Use generated Dart to verify actual constraints, nullability, action output names, API call code, and custom action signatures.
- After each push, run `flutterflow ai refresh-context <project-id>`.

## Validation And Push

Recommended sequence:

```bash
dart test
flutterflow ai validate dsl/edit.dart --project-id <project-id>
flutterflow ai run dsl/edit.dart --project-id <project-id> --commit-message "<summary>"
flutterflow ai refresh-context <project-id>
```

If a generated custom action is changed, run targeted analysis:

```bash
cd generated_code
dart analyze lib/custom_code/actions/<action_file>.dart
```

## Schema Dumps

- Never patch database dumps directly.
- If a DB change is needed, create a standalone SQL proposal/migration file.
- Treat schema dump files as reference-only.

## Components

- Page edits cannot reliably reach inside component internals.
- Prefer editing component params, page-level wrappers, or provide FlutterFlow UI steps for component internals.
- If a component has to be changed with CLI, inspect it directly with `--component`.

## Schema Update Warning

Avoid using FlutterFlow Supabase "Update Schema" casually. It can break widget-to-table bindings for CLI validation. If metadata is corrupted, initialize a fresh workspace against the project and compare context.

