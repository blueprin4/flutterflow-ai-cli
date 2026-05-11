# FlutterFlow AI CLI Field Notes

Version: 0.1.0  
Updated: 2026-05-11

Change log:
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

