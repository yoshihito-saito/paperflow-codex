# RefFlow ChatGPT Workflow

This workflow splits RefFlow into two roles:

- Codex reads the project repository and writes only `refflow/<request-slug>/request.md`.
- ChatGPT treats that `request.md` as the contract, then performs literature scan, paper summaries, review, and proposal writing.

ChatGPT writes results as Markdown in the target project repository and commits those files to GitHub. ChatGPT Desktop or Web UI is not expected to write directly to the local filesystem. Local Obsidian vaults sync by pulling the GitHub commits.

## Standard Layout

```text
<project-repo>/
  refflow/
    <request-slug>/
      request.md
      run-log.md
      run-manifest.yaml
      literature_master.md
      literature_add_candidates.md
      summaries/
        <year>-<first-author>-<short-title>.md
      reviews/
        review.md
      proposals/
        YYYY-MM-DD-<topic>.md
```

## Role Split

### Codex

Codex should inspect the target project repository and generate:

```text
refflow/<request-slug>/request.md
```

The request must name the question, source files to read, literature scope, constraints, and desired output. After that, Codex stops unless explicitly asked to revise the request.

### ChatGPT

ChatGPT reads:

```text
refflow/<request-slug>/request.md
```

Then ChatGPT creates or updates:

- `run-manifest.yaml`
- `run-log.md`
- `literature_master.md`
- `literature_add_candidates.md`
- `summaries/*.md`
- `reviews/review.md`
- `proposals/YYYY-MM-DD-<topic>.md`

ChatGPT should commit the Markdown outputs to the GitHub repository. The local Obsidian vault receives the work through `git pull`.

## Core Rules

- Treat `request.md` as the only contract.
- Do not reread the whole project repository each run.
- Prioritize the source files explicitly listed in `request.md`.
- Do not edit source code, configs, data, or experiment files unless explicitly requested.
- Do not automatically edit Paperpile, Zotero, Google Drive, or other upstream libraries.
- If a shared bibliography such as `paperpile.bib` or `references.bib` is available, use it as a reference source only; do not regenerate it wholesale.
- If Paperpile has a matching PDF for a candidate paper, read that PDF before web sources.
- If a useful paper has no PDF, record it in `literature_add_candidates.md`.
- Classify candidate papers as `core`, `supporting`, `background`, or `exclude`.
- Process one paper at a time. Write the summary immediately after reading that paper.
- Core paper summaries must not be short bullet lists.
- Summaries are persistent memory. Use existing summaries to save context.
- Review major claims must be traceable to per-paper summaries.
- Do not use a paper as evidence in the review unless it has a summary.
- Mark unsupported claims as `missing evidence`.
- Distinguish `direct evidence`, `close analog`, `methodological support`, `theory/background`, and `speculation`.
- When equations, definitions, algorithms, objective functions, or proofs matter, record them explicitly in the summary.
- Reopen original papers during review or proposal writing only when a summary is insufficient or a claim needs verification.

## ChatGPT Prompt Sequence

Use these prompts in order:

1. `prompts/01-read-request.md`
2. `prompts/02-literature-scan.md`
3. `prompts/03-summarize-one-paper.md`
4. `prompts/04-write-review.md`
5. `prompts/05-write-proposal.md`
6. `prompts/06-update-run-log.md`

Repeat prompt 3 for each paper. Do not move to review writing until the relevant per-paper summaries exist.

## GitHub Handoff

1. Codex commits or pushes `refflow/<request-slug>/request.md`, or the user makes it available in the GitHub repository.
2. ChatGPT reads `request.md` through the GitHub repository.
3. ChatGPT creates or updates the RefFlow Markdown artifacts.
4. ChatGPT commits the files to the same GitHub repository.
5. The local machine runs `git pull`.
6. Obsidian refreshes the project vault and shows the new Markdown files.

Suggested commit shape:

```text
Add RefFlow summaries for <request-slug>
Update RefFlow review for <request-slug>
Update RefFlow proposal for <request-slug>
```

## Obsidian Setup

Open the project repository itself as an Obsidian vault. This keeps the research notes beside the code, experiments, and request that define their meaning.

Recommended approach:

- Put RefFlow outputs inside `refflow/<request-slug>/`.
- Use the project repository as the vault root.
- Sync by `git pull` after ChatGPT commits to GitHub.
- Link summaries from other notes with paths such as `[[refflow/<slug>/summaries/2024-smith-short-title]]`.
- Prefer project-local RefFlow folders over one central vault that aggregates all projects. A central vault loses source context and makes GitHub handoff harder to audit.

You may commit `.obsidian/` if the team wants shared vault settings. For most research/code repositories, keep personal workspace state out of git and commit only durable Markdown notes.

Recommended `.gitignore`:

```gitignore
.obsidian/workspace*
.obsidian/cache
.trash/
.DS_Store
```

## Completion Check

A ChatGPT RefFlow run is incomplete if:

- `request.md` was not read first.
- `run-manifest.yaml` or `run-log.md` is missing.
- Literature candidates were not classified.
- A review cites a paper that has no summary.
- Major claims are not traceable to summaries.
- Missing evidence is hidden instead of marked.
- A core paper summary is only a short bullet list.
- Created or updated Markdown was not committed to GitHub.
