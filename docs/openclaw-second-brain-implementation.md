# OpenClaw Closed-Loop Second-Brain Pipeline — Implementation Plan

## Injected user context (verbatim)
- Existing Obsidian vault using strict PARA structure (Projects / Areas / Resources / Archives) with daily notes and MOC system already live.
- Google Business Workspace (full admin access for Drive API, OAuth2 service account possible).
- Local attached WD My Book drive (Windows path will be provided later; treat as primary cold storage for raw media/files >50 MB).
- Fleet: Bob-Main-PC (i5-13600K + RTX 4070 Ti 12 GB + 64 GB RAM) is the heavy lifter; Bev-Main-PC and 29Neibolt-Ubuntu-PC available for lighter tasks or edge inference.
- OpenRouter API key available for mixed-model routing (prefer small/fast local models where possible).
- OpenClaw is already installed and configured on Bob-Main-PC with your preferred models, channels, security hardening, and Obsidian workspace linked.
- Goal: library-grade organization from day one. Ingest volume starts low but must scale cleanly to future NAS (TrueNAS Scale or equivalent).

## 1) Install layout (native OpenClaw skill folders)

Expected target on Bob-Main-PC:
- `~/.openclaw/skills/second-brain-researcher/`
- `~/.openclaw/skills/second-brain-builder/`

### Option A — local copy
```bash
mkdir -p ~/.openclaw/skills
cp -r /path/to/Heuristic/skills/second-brain-researcher ~/.openclaw/skills/
cp -r /path/to/Heuristic/skills/second-brain-builder ~/.openclaw/skills/
```

### Option B — from GitHub repo (recommended)
```bash
git clone https://github.com/CloudJMK/Heuristic.git ~/Heuristic
mkdir -p ~/.openclaw/skills
cp -r ~/Heuristic/skills/second-brain-researcher ~/.openclaw/skills/
cp -r ~/Heuristic/skills/second-brain-builder ~/.openclaw/skills/
```

### Optional bootstrap from project-nomad fork
```bash
git clone https://github.com/Crosstalk-Solutions/project-nomad.git ~/project-nomad-base
# cherry-pick/reuse only reusable scaffolding scripts or watcher adapters
```

## 2) Runtime config checklist (Bob-Main-PC)
1. Define Inbox endpoints in environment or OpenClaw workspace config:
   - `OBSIDIAN_INBOX_PATH`
   - `GDRIVE_INBOX_FOLDER_ID`
   - `WD_INBOX_PATH`
2. Set binary threshold:
   - `BINARY_THRESHOLD_MB=50`
3. Configure Drive service account + restricted scopes (`drive.file` preferred).
4. Configure watch interval and lock/audit directories.

## 3) OpenRouter routing rules
Use local-first fallback chain:
1. Builder classification / lightweight transforms:
   - primary: local 7B/8B instruct model
   - fallback: OpenRouter mid-tier reasoning model
2. Researcher summarization / extraction:
   - primary: local mid-size model if latency acceptable
   - fallback: OpenRouter high-quality model for long-context sources
3. Hard caps:
   - per-item token limit
   - per-day spend ceiling
   - route to low-cost model for retries and boilerplate transforms

## 4) Security model
- Principle of least privilege per skill:
  - Researcher: browse/search/scrape + write to Inbox only.
  - Builder: read Inbox + write PARA + Drive file ops + no arbitrary internet browsing.
- Secrets handling:
  - API keys only in env/secret manager.
  - Redact tokens from logs and notes.
- Provenance:
  - checksum all raw artifacts.
  - immutable ingest audit log.
- Execution safety:
  - never execute downloaded code.
  - quarantine suspicious MIME/signature mismatches.

## 5) Error handling and resilience
- Idempotency key: `sha256 + source_uri`.
- Canonicalization key: normalized URI + hash for dedupe consistency.
- Retry policy: exponential backoff for transient failures.
- Dead-letter queues:
  - `Inbox/_errors`
  - `Inbox/_quarantine`
  - `Inbox/_triage`
- Partial failure strategy:
  - continue queue, emit structured incident note, preserve originals.

## 6) Phased rollout

### Phase 0 — Dry run
- Disable continuous mode.
- Run 10 one-shot captures across mixed content types.
- Validate metadata completeness and link integrity.

### Phase 1 — Controlled automation
- Enable scheduled Researcher runs (low cadence).
- Enable Builder watcher only for Obsidian Inbox.
- Weekly review of `<85 confidence` queue.

### Phase 2 — Full hybrid watchers
- Add Google Drive + WD watchers.
- Turn on deduplication and mirror policies.
- Start metrics dashboard (ingest count, error rate, placement drift).

### Phase 3 — NAS-ready abstraction
- Move WD/Drive paths behind logical storage aliases.
- Prepare migration map for future TrueNAS Scale mount points.

## 7) Iteration protocol (how we refine together)
Per iteration round:
1. You provide one directive or one failing test case.
2. Update either Researcher or Builder skill artifact(s).
3. Run focused test set (unit scenario + end-to-end scenario).
4. Report pass/fail + diff + next risk.

Required test cases each round:
- `TC1`: small text article => Resources note + MOC link.
- `TC2`: 120MB video => WD storage + Obsidian pointer note.
- `TC3`: duplicate URL with changed title => dedupe by canonical/hash.
- `TC4`: ambiguous project relevance => confidence drop and review queue.
- `TC5`: API rate-limit response => retry/backoff + incident note.

## 8) Risk register and mitigation
1. API rate limits
   - Mitigation: adaptive pacing, canonical-URL cache, retry/backoff+jitter, provider fallback.
2. Storage quotas (Drive/local)
   - Mitigation: size threshold routing, 85% warning threshold, retention policies, periodic quota checks.
3. Data duplication
   - Mitigation: canonical URL normalization + content-hash idempotency keys + duplicate index registry.
4. Model drift/inconsistent classification
   - Mitigation: frozen rubric, regression test corpus, confidence thresholding, autoplace freeze on gate failure.
5. Legal scraping risk
   - Mitigation: robots/TOS-aware allowlist policy, public-data-only posture, explicit legal basis logging in manifest.
6. Permission overreach in OpenClaw
   - Mitigation: per-skill scoped tools, deny-by-default filesystem roots, path traversal/symlink-escape rejection.
