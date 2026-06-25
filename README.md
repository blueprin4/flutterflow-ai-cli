# FlutterFlow AI CLI Field Notes

Version: 0.2.2
Updated: 2026-06-25

Change log:
- 0.2.2 - Added a version-history diffing note (refresh-context loaded-version scope; non-destructive restore and bisecting).
- 0.2.1 - Added action-wiring guidance for grouped local-state updates and generated Dart verification.
- 0.2.0 - Added repeatable `dsl/edit.dart` sync guidance and post-push cleanup prompts for one-off UI/page/component migrations.
- 0.1.1 - Tightened wording around native-first guidance, component edits, and schema refresh cautions.
- 0.1.0 - Initial public release with a Codex skill and battle-tested FlutterFlow AI CLI notes.

This repo contains a community-oriented Codex skill for working with FlutterFlow AI CLI workspaces. It is intentionally limited to reusable workflow guidance and sanitized field notes.

No FlutterFlow project export, generated app code, database dump, API key, Supabase key, or private project metadata belongs in this repo.

## Contents

- `ff-cli/SKILL.md` - the main skill entrypoint and day-to-day workflow rules.
- `ff-cli/references/battle-tested-lessons.md` - detailed lessons and workarounds from real CLI debugging.

## Use

Copy the `ff-cli` folder into your Codex skills directory:

```bash
mkdir -p "$CODEX_HOME/skills"
cp -R ff-cli "$CODEX_HOME/skills/ff-cli"
```

Then start a Codex thread and ask it to use the `ff-cli` skill when working on FlutterFlow AI CLI projects.
