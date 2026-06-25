# Battle-Tested FlutterFlow AI CLI Lessons

Version: 0.2.2
Updated: 2026-06-25

Change log:
- 0.2.2 - Added a version-history diffing note: `refresh-context` reads only the currently-loaded version, and restoring a version is non-destructive.
- 0.2.1 - Added action-wiring guidance for grouped local-state updates and generated Dart verification.
- 0.1.1 - Tightened native-first, component-edit, and schema-refresh guidance.
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

## Action Wiring

- The public DSL helper `SetState('field', value)` compiles to one FlutterFlow local-state action per field. Do not use a long sequence of individual `SetState` actions when the intent is one grouped page/component state update.
- For buttons or handlers that need to assign many page-state fields at once, prefer a single FlutterFlow local-state update action with multiple field updates. If the typed DSL cannot express it, use a narrow `app.raw()` mutation against the existing action chain: merge consecutive `FFAction.localStateUpdate.updates`, dedupe repeated fields, and preserve non-state follow-up actions such as tab/page-view jumps.
- When cleaning up an existing designer-built action chain, mutate only the action chain. Do not rebuild or replace the surrounding page layout just to wire state updates; preserve designer choices such as `Wrap` vs `ListView`.
- After grouping state updates, run `flutterflow ai validate`, push, `flutterflow ai refresh-context`, and inspect generated Dart. The expected generated shape is one block of model assignments followed by the original follow-up action, not repeated `safeSetState(() {})` after every field.

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

Prefer FlutterFlow-native surfaces before custom code. This is partly a maintainability preference and partly a response to observed CLI failure modes: native resources remain editable in FlutterFlow UI, and they avoid several brittle generated-code and raw-proto paths.

Use these surfaces first:
- API Calls for REST/RPC functions.
- Supabase queries/mutations where the CLI validates cleanly; otherwise prefer FlutterFlow UI wiring, API/RPC calls, or action-block wrappers.
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

## Version History And Diffing

- `flutterflow ai refresh-context` captures only the **currently-loaded** project version. To diff two FlutterFlow commits, restore one, run `refresh-context`, and snapshot `generated_code/`; then restore the other, run `refresh-context`, and diff against the snapshot.
- Restoring a FlutterFlow version is non-destructive: it makes a copy of the chosen version the new current one and leaves newer commits in history. You can bisect a regression freely (restore the midpoint, test, halve) without losing work. Watch for large commits hidden behind a small message; they can carry collateral action-chain edits unrelated to the title.

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

- Do not assume a page edit can safely reach inside a component instance and change that component's internals.
- Direct component edits can work. Inspect the component directly with `flutterflow ai inspect <project-id> --component <ComponentName> --dsl-json`, then use `app.editComponent(...)` for component internals.
- From a page, prefer editing the component instance params, page-level wrappers, visibility, or placement.
- If the intended change must remain easy to maintain in FlutterFlow UI and the CLI path is fragile, provide FlutterFlow UI steps instead of forcing raw proto edits.

## Schema Refresh And Validation Regressions

The connected Supabase metadata pattern fixes the known bare `app.supabase(...)` validation failure. That is separate from schema-refresh risk.

If FlutterFlow Supabase "Update Schema" or another metadata refresh is followed by unexpected validation failures in existing table bindings, treat it as a metadata/context regression until proven otherwise:
- Run `flutterflow ai refresh-context <project-id>`.
- Compare a fresh workspace against the existing workspace.
- Inspect generated Supabase table classes and the relevant `dsl_json/` before rewriting working UI.
