# 02 Literature Scan

Use `refflow/<request-slug>/request.md` to find candidate literature. Use existing summaries, local bibliography files, shared bibliography files, and source notes before web search.

## Tasks

- Search existing RefFlow summaries first.
- Check configured bibliography sources, including shared `paperpile.bib` or `references.bib` when available.
- Check Paperpile or local PDF availability when accessible.
- Search web sources only for gaps.
- Add candidates to `literature_master.md`.
- Assign each candidate one priority: `core`, `supporting`, `background`, or `exclude`.
- Record PDF and local availability as known, missing, inaccessible, or unknown.
- Add useful papers without PDFs to `literature_add_candidates.md`.
- Update `run-manifest.yaml` and `run-log.md`.

## Rules

- Do not deep read papers in this step.
- Do not write per-paper summaries yet.
- Do not use a paper as review evidence until a summary exists.
- If Paperpile has a matching PDF, mark it as the preferred full-text source.
- If availability is unknown, say so explicitly.

## Output

Commit updates to `literature_master.md`, `literature_add_candidates.md`, `run-manifest.yaml`, and `run-log.md`.
