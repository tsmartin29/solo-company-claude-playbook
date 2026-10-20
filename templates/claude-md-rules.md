# Rules block for your global CLAUDE.md (edit the numbers to fit your machine)

## Deleting and overwriting
- Never permanently delete user files. Move them to the Trash: `mv <path> ~/.Trash/`. Permanent deletes are only for temp and build areas (/tmp, build folders, node_modules, virtualenvs, caches).
- Before overwriting a tested file, keep a dated copy next to it: `<file>.<YYYY-MM-DD>_<reason>.bak`. Never keep the only copy in /tmp or a scratch folder.
- Never give me a script with `rm -rf`, `dd`, or overwrite-in-place that has a default or guessable target path. Make me type the exact target and echo it back.
- List what will be removed before removing it.

## Heavy work
- One heavy job at a time, machine-wide (over ~1.5 GB or a long CPU run). Workers = 1. Run it through the queue: `heavyrun "<session>: <job>" <command...>`; short small jobs use `HEAVY_LIGHT=1`.
- Label queued jobs with their real peak memory.
- Do not start extra sessions to run heavy work in parallel.

## Finishing
- One task per session. At ~200k tokens or when done, write a handoff note and suggest a fresh session.
- At most 2 sub-agents at once; small/mid models for mechanical work.
- Before saying built, fixed or done, use the result like a real user would and say what you tried and what you saw. Never take over my screen or keyboard for tests.
- When something is blocked, say what we would need AND the cheapest workaround.
- Delete the scratch and build folders you created once the task is verified. Never touch other sessions' work in progress.
