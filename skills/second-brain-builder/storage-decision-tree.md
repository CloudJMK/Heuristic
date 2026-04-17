# Storage Decision Tree (Builder)

Executed during **Step 4** of the Builder workflow. All file operations are MCP tool calls. Storage routing is **size-based and type-based only** — there is no runtime quota check because `gdrive` is a filesystem-backed server without Drive API access. If a write fails, handle it in the failure branch.

## Server → path reference

| Server | Root path |
|--------|-----------|
| `obsidian` | `/mnt/d/29Neibolt_Drop` |
| `gdrive` | `/mnt/g/Shared drives/CloudPacts` |
| `filesystem` | WD/MyBook Inbox + audit log |

---

## Decision flow

```
START
│
├─ Always: record checksum from manifest. Compute idempotency key:
│    ikey = sha256(content) + canonical_source_uri
│
├─ Is file_size_mb > binary_threshold_mb (default 50)?
│
│   YES — Large binary
│   ├─ filesystem:write_file(wd_path + artifact)
│   ├─ obsidian:write_note(para_path + pointer_note.md)
│   │     # pointer contains: title, WD path, SHA256, provenance block
│   └─ DONE
│
│   NO — continue
│
├─ What is content_type?
│
│   CASE text_note
│   ├─ obsidian:write_note(para_path + note.md)
│   │     # full markdown content in note body
│   └─ DONE
│
│   CASE pdf_doc | media_asset | dataset
│   ├─ gdrive:write_file(cloudpacts_path + artifact)
│   │     ON FAILURE:
│   │     ├─ filesystem:write_file(wd_path + artifact)      # overflow to WD
│   │     └─ obsidian:write_note(00-Inbox/_alerts/ + alert-<date>.md)
│   │           # alert: failed path, timestamp, recommended action
│   ├─ obsidian:write_note(para_path + index_note.md)
│   │     # index note: CloudPacts path (or WD path on overflow),
│   │     #             title, ≤200-word summary, provenance block
│   └─ DONE
│
│   CASE code_repo | binary_other
│   ├─ filesystem:write_file(wd_path + artifact)
│   ├─ obsidian:write_note(para_path + index_note.md)
│   │     # index note: repo slug, commit/release hash, source URI, provenance
│   └─ DONE
│
│   CASE unknown / unresolved
│   ├─ Set needs_review: true
│   ├─ obsidian:write_note(04-Resources/ + note.md)
│   │     # default PARA: Resources
│   └─ DONE
```

---

## Required fields in every output note

Every branch must produce at least one Obsidian note written via `obsidian:write_note`:

```yaml
title: <canonical title>
summary: <≤200 words>
para_primary: Projects|Areas|Resources|Archives
para_path: <full hierarchical path>
backlinks: []
provenance:
  source_uri: <url>
  hash_sha256: <hex>
  ingest_utc: <ISO-8601>
  storage_location: obsidian|gdrive|wd
needs_review: <true|false>
```

## Retention rules
- Never call any delete tool on original artifacts.
- Duplicate binaries: flag `needs_review: true`; human decides retention.
- Alert notes in `00-Inbox/_alerts/` accumulate with date-suffixed filenames — do not overwrite.
