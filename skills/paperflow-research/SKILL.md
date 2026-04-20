---
name: paperflow-research
description: Use when Codex is asked to answer a source request page with literature-grounded research, check the user's existing paper library when available, search free web sources for missing papers, create per-paper summaries, synthesize a scoped review, and return an answer plus concrete proposals.
---

# Paperflow Research

## Core Principle

Treat literature-grounded answering as a reproducible request-response pipeline, not a one-shot answer. The main input is a source request page: a Markdown page from a source repository, notes folder, or user-provided path that states a question, source context, relevant files, and desired output.

Preserve intermediate files inside the source repository: copied/normalized request, per-paper summaries, literature master, scoped review, answer, proposal, run log, and literature add candidates.

Use the request page as authoritative. Read only the source files named in the request unless the request is underspecified or the user asks for broader inspection. Source context extraction supports the answer; it is not the primary goal.

Store generated research files in the source repository by default, under `paperflow/<request-slug>/`. Treat `paperflow-codex` as the skill repository, not as the place where each project's answers live.

## One-Line Invocation

When the user says something like this, run the full workflow:

```text
Use paperflow-research to answer /path/to/source/request.md.
```

If no output location is specified, save the results in the same source repository under `paperflow/<request-slug>/`.

## Standard Workflow

1. Read the source request page, or create one from `references/source-request-template.md` if the user gives the request in chat.
2. Extract the request slug, source repository or source page path, research question, source context, named source files, literature scope, library search terms, and desired answer format.
3. Create or update the output folder in the source repository: `paperflow/<request-slug>/`.
4. Create or update `run-manifest.yaml`, `run-log.md`, `literature_master.md`, and `literature_add_candidates.md`.
5. Read only the source files named in the request. If no files are named and the source context is insufficient, do a narrow fallback read of `README*`, cited docs, or obvious config/model files.
6. Check available paper-library sources first. Use Google Drive / Paperpile when available; otherwise check local bibliography and notes files in the source repository.
7. Reuse existing metadata, notes, summaries, PDFs, and BibTeX entries when available.
8. Deduplicate papers before web search using DOI, arXiv ID, PMID, then normalized title.
9. Search free web sources only for gaps: arXiv, PubMed, OpenAlex, Crossref, publisher pages, author pages, GitHub, and project pages.
10. Process papers one at a time. After reading each paper beyond metadata level, immediately create or update its Markdown summary before opening the next paper.
11. Build or update `literature_master.md` from the paper summaries.
12. Synthesize a scoped review for the request.
13. Produce an answer that directly responds to the request, with links to the supporting files.
14. Produce a proposal with concrete experiments, implementation changes, or decision points when the request asks for next steps.
15. Update `run-manifest.yaml` and `run-log.md`.
16. Suggest papers to add to the user's library, but do not add, upload, move, rename, or delete files without explicit user approval.

## Source Request Handling

The source request page is the contract. It should contain:

- the question to answer
- the source repository or source page path
- the relevant source context
- the source files to read
- the literature scope and library search terms
- the desired output
- constraints and claim boundaries

Use `references/source-request-template.md` for new requests. If the user supplies a source page from another repository, keep the original request in place and copy or normalize it into the output folder only for record keeping:

`paperflow/<request-slug>/request.md`

Do not edit the original source page unless the user explicitly asks.

## Source Context Extraction

Source context extraction is narrow and request-guided. Start with files named in the source request, such as:

- a design note or issue page
- `README*`
- experiment notes, notebooks, or logs
- model, environment, dataset, and training code
- configuration files

If the request already contains enough context, source file reads may be minimal. If the request is underspecified, read the smallest set of source files needed to answer accurately, then record what was inspected in the answer and proposal.

## Existing-Library-First Literature Search

Use the user's existing paper library first when one is available. Paperpile / Google Drive is optional, but when it is available it has priority over local bibliography files and web search.

Recommended order:

1. If Google Drive / Paperpile is available, search it first.
2. If Google Drive / Paperpile is unavailable, record that in `run-log.md` and continue.
3. Search local source-repository files such as `references.bib`, `*.bib`, `literature.md`, `notes/`, `docs/`, `paperflow/`, and prior summaries.
4. Search free web sources for the remaining gaps.
5. Record each paper's library status as one of: `in Paperpile`, `in local bibliography`, `in source notes`, `not found locally`, or `unknown`.
6. Record each paper's PDF status as one of: `PDF available`, `PDF missing`, `open-access PDF found`, `metadata only`, or `unknown`.
7. If a relevant paper has no local PDF, explicitly mark `PDF missing` in its summary and add it to `literature_add_candidates.md` with a suggestion to download or add the PDF.

## Web Search Rules

Use web search after checking available local/library sources. Prefer stable, free sources:

- arXiv
- PubMed
- OpenAlex
- Crossref
- publisher landing pages
- author pages
- official project pages
- GitHub repositories linked by the paper

Avoid paid APIs, paywalled databases, or subscription-only sources unless the user explicitly requests them.

For each web result, separate metadata-level knowledge from content that has actually been read. Mark unread or partially read papers clearly.

## Research Depth And Token Budget

Use `standard` depth by default unless the request specifies otherwise:

- `quick`: scan up to 20 candidate papers, summarize up to 5 papers, deeply read up to 2 papers.
- `standard`: scan up to 40 candidate papers, summarize up to 20 papers, deeply read up to 10 papers.
- `deep`: scan 50 or more candidate papers when useful, summarize up to 25 papers, deeply read up to 15 papers.

Keep the context window lean:

- Do not batch-read many full papers into the same context.
- Read one paper, write or update its summary, then move to the next paper.
- Treat saved summaries as the primary input for the review, answer, and proposal.
- Re-open original papers only when a claim needs verification or a summary is insufficient.
- Prefer concise summaries over copying long passages from papers.

## Mathematical And Algorithmic Accuracy

When a paper's contribution depends on equations, definitions, objectives, update rules, architectures, or algorithmic assumptions, verify those details from the paper before adding them to the summary.

- Preserve important equations in concise LaTeX form when they are needed for the review or proposal.
- Record symbol meanings, dimensions, assumptions, loss terms, constraints, and equation numbers when available.
- Distinguish exact formulas from paraphrased intuition.
- If an equation, derivation step, or notation is uncertain, mark it as `needs verification` instead of guessing.
- For algorithms, capture the inputs, outputs, core steps, and any stated complexity or convergence conditions when relevant.

## Per-Paper Summaries

Create or update one Markdown file for every paper that is read beyond metadata level. Use:

`paperflow/<request-slug>/summaries/<year>-<first-author>-<short-title>.md`

Use `references/summary-template.md` as the template. Each summary must preserve citation metadata, source links, library status, PDF status, one-sentence takeaway, methods, key findings, mathematical or algorithmic details when relevant, limitations, relationship to other papers, relevance score, and notes for the final review.

Do not overwrite an existing summary casually. If it exists, update it by preserving useful prior notes and adding new evidence, with a short `Update Notes` section when appropriate.

## Review, Answer, And Proposal

Generate the review, answer, and proposal only after the relevant per-paper summaries exist. Use:

- `references/review-template.md` for literature synthesis.
- `references/answer-template.md` for the request answer.
- `references/proposal-template.md` for project diagnosis and next steps.
- `references/run-manifest-template.yaml` for run metadata.

The answer should be the user-facing result. It should include:

- direct answer
- confidence and scope
- source context used
- literature evidence
- synthesis of mechanisms
- concrete recommendations
- links to summaries, review, and proposal

The proposal should connect literature to the source repository. It should include:

- source context used
- key literature themes
- likely gaps or failure modes
- concrete hypotheses
- actionable experiments or implementation changes
- risks and validation checks
- literature add candidates

## Output Layout

Use this source-repository structure unless the user requests otherwise:

```text
<source-repo>/
  paperflow-requests/
    <request-slug>.md
  paperflow/
    <request-slug>/
      request.md
      answer.md
      run-manifest.yaml
      run-log.md
      literature_master.md
      literature_add_candidates.md
      summaries/
      reviews/
        review.md
      proposals/
        YYYY-MM-DD-<topic>.md
```

Keep generated output files inside the source repository so the answer travels with the code, notes, and experiments it explains.

## Continuing Prior Local Work

When a user asks to continue a prior answer in the same repository:

1. Read the source request page.
2. Check `paperflow/<request-slug>/run-manifest.yaml`.
3. Read the existing answer and supporting files as needed:

```text
paperflow/<request-slug>/
  answer.md
  literature_master.md
  run-log.md
  summaries/
```

## Safety

- Do not add PDFs or metadata to Paperpile, Zotero, BibTeX files, or any other library automatically.
- If a useful paper has no local PDF, suggest downloading or adding it, but do not download, upload, or move it without explicit approval.
- Do not upload, delete, rename, or move Google Drive files without explicit approval.
- Do not modify source code, configs, data, or experiment files unless the user explicitly requests implementation. Writing `paperflow/<request-slug>/` output files is allowed as part of this skill.
- Be clear when a paper was found but not fully read.
- Use exact citations, DOI, arXiv ID, or URLs whenever available.
