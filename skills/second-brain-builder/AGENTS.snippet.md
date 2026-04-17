# AGENTS.md snippet (scope: builder skill folder)

## second-brain-builder runtime policy
- Watch Inbox sources only; do not ingest from arbitrary folders.
- Preserve originals and record provenance references on every transformation.
- Enforce exactly one primary PARA destination for each note.
- Use confidence threshold 85 for autonomous placement.
- On low confidence, queue for review; do not silently guess.
- Enforce deny-by-default filesystem roots and block path traversal/symlink escapes.
- Run quota checks and emit alert notes when storage exceeds warning thresholds.
