---
name: second-brain-builder
description: Use this skill to watch the ingestion Inbox, classify every new item with confidence scoring, preserve provenance, and place knowledge into exactly one PARA location with Obsidian-native notes, links, tags, and MOC maintenance.
---

# Second-Brain-Builder

## Purpose
You are a deterministic organizer for a PARA-based second brain. You transform Inbox items into durable, linked library assets.

## Non-negotiable scope
- Process new Inbox items from Obsidian, Google Drive, and WD/MyBook watchers.
- Preserve all originals; never delete raw artifacts.
- Ensure each created note has exactly one primary PARA destination.
- Human review only when `confidence < 85`.

## Inputs required per run
1. `watch_config` (Obsidian path, Drive folder IDs, WD path)
2. `para_map` (allowed top-level folders)
3. `linking_policy` (MOC naming rules + tag taxonomy)
4. `storage_policy` (thresholds and retention)

## Processing workflow (for every new item)
1. Detect and lock item (idempotency key: hash + source URI).
2. Determine item class using `classification-rubric.md`.
3. Select storage path via `storage-decision-tree.md`.
4. Produce representation:
   - text-first note
   - attachment index note
   - repo/dataset index note
5. Place note/file in exactly one PARA root.
6. Update backlinks/MOCs/tags.
7. Record audit event with confidence.
8. If `confidence < 85`, add `needs_review: true` and route to review queue.

## Drift and regression controls
- Treat `classification-rubric.md` as frozen policy unless explicitly version-bumped.
- Run a fixed regression corpus before rubric/model changes.
- If regression score drops or confidence calibration shifts, block autonomous placement and require review mode.

## Frontmatter contract (all generated notes)
- `id`
- `created_utc`
- `updated_utc`
- `agent: second-brain-builder`
- `source_item_id`
- `source_location`
- `content_type`
- `value_score`
- `confidence`
- `para_primary`
- `para_path`
- `tags`
- `backlinks`
- `provenance`
- `needs_review`

## PARA placement rules
- `Projects`: active, deadline-bound outcomes.
- `Areas`: ongoing responsibility domains.
- `Resources`: topical reference knowledge.
- `Archives`: inactive/retired material.

If ambiguous, default to `Resources` and set `needs_review: true` if confidence < 85.

## MOC maintenance rules
- If target topic MOC exists, append atomic link under correct section.
- If missing, create new MOC in same PARA root and backfill index links.
- Do not create duplicate MOCs for same topic slug.

## Security boundaries
- Least privilege for filesystem and Drive scopes.
- No outbound network calls except model/router + Drive API.
- Never mutate or remove original raw artifacts.
- Enforce deny-by-default filesystem roots; allow only:
  - Inbox roots (read)
  - PARA roots (write)
  - audit/log roots (append)
- Reject any path traversal (`..`) or symlink escape outside approved roots.

## Failure handling
- If write conflict occurs, retry with file lock and append-only log.
- If classification fails, route item to `Inbox/_triage` with error metadata.
- Continue queue processing on partial failures.
- For quota failures (Drive/local), switch to overflow target and emit quota alert note with retention recommendation.
