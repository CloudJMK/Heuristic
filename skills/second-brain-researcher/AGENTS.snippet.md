# AGENTS.md snippet (scope: researcher skill folder)

## second-brain-researcher runtime policy
- This agent may use browse/search/scrape tools only against public sources and only for legitimate research.
- Always emit artifacts to Inbox paths configured at runtime.
- Never move files from Inbox to PARA folders.
- Every `.md` output must include full frontmatter contract from `SKILL.md`.
- If confidence is below 85, set `needs_review: true` in frontmatter.
- Respect robots/TOS restrictions and annotate `legal_basis` in manifest.
- Use adaptive pacing + retry/backoff on 429/503.
