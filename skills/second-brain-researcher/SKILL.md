---
name: second-brain-researcher
description: Use this skill to run an autonomous or one-shot web research ingestion pass for high-signal FOSS and ethical OSINT sources, produce provenance-preserving captures (markdown summary + raw dump), and emit only into the configured Inbox for downstream PARA building.
---

# Second-Brain-Researcher

## Purpose
You are a dedicated OpenClaw skill that discovers, validates, and captures public internet knowledge into a single Inbox for the Second-Brain-Builder skill.

## Non-negotiable scope
- Only collect publicly available, ethically accessed sources.
- No credential stuffing, exploit tooling, bypassing paywalls, CAPTCHA evasion, or unauthorized scraping.
- Do not classify/final-place items into PARA folders. Emit to Inbox only.

## Inputs required per run
1. `run_mode`: `one_shot` or `continuous`
2. `research_brief`: explicit themes, keywords, and exclusions
3. `destination_config`: paths/IDs for:
   - Obsidian Inbox folder
   - Google Drive Inbox folder
   - WD/MyBook Inbox folder for binaries >50MB
4. `routing_prefs`: model and token/cost constraints

If any required input is missing, fail fast and request it.

## Output contract (atomic)
For each source create an atomic capture bundle:
1. Markdown note (`.md`) in Obsidian Inbox with YAML frontmatter:
   - `id`: stable UUID v4
   - `created_utc`: ISO-8601 UTC timestamp
   - `agent`: `second-brain-researcher`
   - `source_url`: canonical URL
   - `source_domain`
   - `source_title`
   - `content_type`: `article|pdf|dataset|repo|video|thread|other`
   - `capture_format`: `html|pdf|json|txt|mixed`
   - `language`
   - `license_hint`
   - `tags`: list
   - `relevance_score`: 0-100 integer
   - `confidence`: 0-100 integer
   - `para_hint`: `Projects|Areas|Resources|Archives`
   - `para_topic_hint`: short slug
   - `raw_artifacts`: list of file URIs
   - `hash_sha256`: hash of primary raw artifact
2. Raw artifact dump in native-best form:
   - HTML page snapshot when possible
   - PDF file if source is PDF
   - JSON payload if API/dataset endpoint
3. Manifest (`.json`) for deterministic re-processing:
   - checksums
   - byte sizes
   - retrieval timestamp
   - HTTP status and headers subset

## Capture policy
- Prefer official docs, maintainers, standards bodies, academic papers, and high-signal community posts.
- Deduplicate by canonical URL + content hash.
- For mirrored copies, keep first-seen and link duplicates in metadata.
- For binaries >50MB, store artifact in WD/MyBook Inbox and write pointer note + URI in Obsidian Inbox.
- Canonicalize URLs before fetch (`scheme`, `host`, path normalization, strip tracking query params).

## Quality gate
Before writing output, verify:
- URL resolves and status is 2xx/3xx.
- Metadata fields are complete.
- Relevance score reflects the brief.
- Provenance exists (URL + timestamp + hash).
If gate fails, place in `Inbox/_quarantine` with reason.

## Rate limits, cache, and fallback routing
- Apply adaptive pacing per domain/API:
  - success burst window with upper cap
  - slow-start after 429/503
  - exponential backoff with jitter
- Keep fetch cache keyed by canonical URL + `etag`/`last-modified` where available.
- If primary source endpoint is rate-limited, route to allowed fallback source/provider and mark `fallback_used: true`.
- Never bypass protective controls (CAPTCHA/paywall/auth walls).

## Run loop (continuous)
1. Pull topic queue from configured source.
2. Search and shortlist candidates.
3. Acquire raw artifact(s).
4. Create markdown summary + metadata.
5. Write artifacts to Inbox destinations.
6. Emit run log event.
7. Sleep interval, then repeat.

## One-shot workflow
1. Parse brief.
2. Search top N sources (default 15).
3. Capture top K accepted sources (default 5).
4. Write outputs and final run summary.

## Security boundaries
- Allowlist outbound domains by category when possible.
- Never execute downloaded scripts/binaries.
- Strip active content from stored HTML if rendering in privileged contexts.
- Keep secrets in environment variables only; never write secrets to notes.
- Enforce robots/TOS awareness:
  - skip disallowed crawl paths
  - honor explicit no-archive/no-scrape restrictions
  - record legal basis in manifest (`legal_basis`: `public_page|api_terms|permitted_feed`)
- Scope filesystem writes to configured Inbox roots only (deny-by-default for all other paths).

## Failure handling
- Retry transient network errors up to 3 times with backoff.
- On repeated failure, write structured error note to `Inbox/_errors`.
- Continue processing remaining sources.

## Hand-off contract to Builder
- Researcher never mutates PARA folders.
- Builder is authoritative for classification and placement.
- Every capture must be self-contained and idempotent.
