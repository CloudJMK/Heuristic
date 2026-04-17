# Storage Decision Tree (Builder)

1. Is item raw/original artifact from Inbox?
   - Yes: preserve untouched and record checksum.
   - Also compute canonical identity key: `sha256 + canonical_source_uri`.
2. Is file size > 50MB?
   - Yes: store raw in WD My Book path, create Obsidian pointer note, optional Drive mirror.
   - No: continue.
3. Are storage quotas near threshold?
   - If Drive > 85% full: route eligible binaries to WD first.
   - If WD > 85% full: route eligible files to Drive first.
   - Emit quota alert + retention candidates list.
4. Is type pure text or easily text-normalized?
   - Yes: convert to clean markdown in chosen PARA folder.
5. Is type PDF/media/dataset?
   - Yes: keep artifact in Drive (or WD if large), create index/attachment note in PARA folder.
6. Is type repo or large binary bundle?
   - Yes: store under archival/code location, create index note with purpose, commit hash/release tag, provenance links.
7. If unsure after steps 1-6:
   - Place note in Resources with `needs_review: true` when confidence <85.

## Linking outputs
- Every placement must produce at least one Obsidian note containing:
  - canonical title
  - summary (<=200 words)
  - PARA primary path
  - links/backlinks to related notes and MOCs
  - provenance block (source URI, checksum, ingest timestamp)

## Retention baseline
- Never delete originals automatically.
- Mark duplicate binaries for human-reviewed retention only.
- Prefer deduplicated hardlink/reference pointers where filesystem supports it.
