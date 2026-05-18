# 03 Summarize One Paper

Process exactly one paper. Use this prompt once per paper.

## Input

- Request: `paperflow/<request-slug>/request.md`
- Candidate row from `literature_master.md`
- Full text source, preferably a Paperpile or local PDF when available
- Template: `chatgpt-workflow/templates/summary.md`

## Tasks

- Read the selected paper only.
- Write a summary to `paperflow/<request-slug>/summaries/<year>-<first-author>-<short-title>.md`.
- Include citation metadata, PDF/local status, read status, evidence strength, request relevance, key results, limitations, and source locations.
- Record mathematical or algorithmic details when relevant, including equations, definitions, objectives, proofs, or algorithms.
- Update `literature_master.md`, `run-manifest.yaml`, and `run-log.md`.

## Rules

- Do not batch-read multiple papers.
- Do not write the review in this step.
- A `core` paper summary must include enough detail to support later synthesis.
- If the request depends on the whole paper, include section-level notes for the whole relevant paper.
- If evidence is weak or incomplete, mark it clearly.

## Output

Commit the summary and updated tracking files to GitHub.
