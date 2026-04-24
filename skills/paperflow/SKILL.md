---
name: paperflow
description: Use when Codex is asked to answer a source request page with literature-grounded research, check the user's existing paper library when available, search free web sources for missing papers, create detailed per-paper summaries, synthesize a scoped review, and return an answer plus concrete proposals.
---

# Paperflow

## Core Principle

Treat literature-grounded answering as a reproducible request-response pipeline, not a one-shot answer. The main input is a source request page: a Markdown page from a source repository, notes folder, or user-provided path that states a question, source context, relevant files, and desired output.

Preserve durable research artifacts inside the source repository: copied or normalized request, per-paper summaries, scoped review, run manifest, run log, and literature add candidates. Return the final answer and proposal in chat by default instead of creating standalone `answer.md`, `proposal.md`, `literature_master.md`, or `source_context.md` files unless the user explicitly asks for them.

Use the request page as authoritative. Read only the source files named in the request unless the request is underspecified or the user asks for broader inspection. Source context extraction supports the answer; it is not the primary goal.

Store generated research files in the source repository by default, under `paperflow/<request-slug>/`. Treat `paperflow-codex` as the skill repository, not as the place where each project's answers live.

## One-Line Invocation

When the user says something like this, run the full workflow:

```text
Use paperflow to answer /path/to/source/paperflow/<request-slug>/request.md.
```

If no output location is specified, save the results in the same source repository under `paperflow/<request-slug>/`.

## Standard Workflow

1. Read the source request page, or create one at `paperflow/<request-slug>/request.md` from `references/source-request-template.md` if the user gives the request in chat.
2. Extract the request slug, source repository or source page path, research question, source context, named source files, literature scope, library search terms, and desired answer format.
3. Create or update the output folder in the source repository: `paperflow/<request-slug>/`.
4. Create or update `run-manifest.yaml`, `run-log.md`, and `literature_add_candidates.md`.
5. Read only the source files named in the request. If no files are named and the source context is insufficient, do a narrow fallback read of `README*`, cited docs, or obvious config/model files.
6. Check available paper-library sources first. Use Google Drive / Paperpile when available; otherwise check local bibliography and notes files in the source repository.
7. Reuse existing metadata, notes, summaries, PDFs, and BibTeX entries when available.
8. Deduplicate papers before web search using DOI, arXiv ID, PMID, then normalized title.
9. Search free web sources only for gaps: arXiv, PubMed, OpenAlex, Crossref, publisher pages, author pages, GitHub, and project pages.
10. Process papers one at a time. After reading each paper beyond metadata level, immediately create or update its Markdown summary before opening the next paper.
11. Synthesize a scoped review for the request from the paper summaries.
12. Produce an answer that directly responds to the request, with links to the supporting files, and return it in chat unless the user explicitly asks for a saved answer file.
13. Produce a proposal with concrete experiments, implementation changes, or decision points when the request asks for next steps, and return it in chat unless the user explicitly asks for a saved proposal file.
14. Do not create standalone `source_context.md` or `literature_master.md` files unless the user explicitly requests persisted copies.
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

Use `references/source-request-template.md` for new requests and save them inside the generated output folder:

`paperflow/<request-slug>/request.md`

If the user supplies a source page from another location, keep the original request in place and copy or normalize it into `paperflow/<request-slug>/request.md` for record keeping.

Do not edit the original source page unless the user explicitly asks.

## Source Context Extraction

Source context extraction is narrow and request-guided. Start with files named in the source request, such as:

- a design note or issue page
- `README*`
- experiment notes, notebooks, or logs
- model, environment, dataset, and training code
- configuration files

If the request already contains enough context, source file reads may be minimal. If the request is underspecified, read the smallest set of source files needed to answer accurately, then record what was inspected in the answer and proposal. Do not create a separate `source_context.md` file unless the user explicitly asks for one.

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
- Read the whole paper section by section when full-paper reading is needed.
- Do not load the entire paper into one model context.
- For each section, write section-level notes before moving to the next section.
- Preserve detailed notes for sections relevant to the request.
- For less relevant sections, write a concise coverage note explaining what was checked.
- The goal is not to skip the paper, but to keep the active context small while preserving full-paper coverage.
- Read one paper, write or update its summary, then move to the next paper.
- Treat saved summaries as durable notes, not as tiny abstracts.
- For review, answer, and proposal synthesis, read the compact sections first and then open detailed sections for high-relevance papers.
- Re-open original papers only when a claim needs verification or the detailed summary is insufficient.
- Avoid copying long passages from papers, but preserve detailed explanations, equations, methods, evidence, limitations, and source-repository relevance in your own words.
- When PDF-to-Markdown or JSON extraction is used, do not pass raw extracted output directly into the model context.
- Clean boilerplate, references, repeated headers or footers, and OCR artifacts first when possible.
- Then process the cleaned paper section by section.
- Use extracted JSON primarily for source locations, equations, figures, and tables when needed.
- Use the cheapest capable model by default for metadata extraction, PDF or Markdown cleanup, section-level notes, ordinary empirical paper summaries, and citation normalization.
- Escalate to a stronger reasoning model only when the paper or section is high relevance, math-heavy, theory-heavy, proof-heavy, algorithmically central, marked `needs verification`, or used as primary evidence in the final review.
- Do not use a stronger reasoning model for raw PDF reading by default.
- Use stronger reasoning only for targeted verification, mathematical or algorithmic sections, and final synthesis when needed.

## Mathematical And Algorithmic Accuracy

When a paper's contribution depends on equations, definitions, objectives, update rules, architectures, or algorithmic assumptions, verify those details from the paper before adding them to the summary.

- Preserve important equations in concise LaTeX form when they are needed for the review or proposal.
- Write important equations directly into the per-paper summary when they matter for the request, not just a prose reference to them.
- Explain each important equation in prose, including symbol meanings, dimensions, assumptions, loss terms, constraints, and equation numbers when available.
- Include term-by-term intuition and why the equation matters for the source request.
- Distinguish exact formulas from paraphrased intuition.
- If an equation, derivation step, or notation is uncertain, mark it as `needs verification` instead of guessing.
- For algorithms, capture the inputs, outputs, core steps, and any stated complexity or convergence conditions when relevant.

## Per-Paper Summaries

Create or update one Markdown file for every paper that is read beyond metadata level. Use:

`paperflow/<request-slug>/summaries/<year>-<first-author>-<short-title>.md`

Use `references/summary-template.md` as the template. Each summary must preserve citation metadata, source links, library status, PDF status, one-sentence takeaway, methods, key findings, mathematical or algorithmic details when relevant, limitations, relationship to other papers, relevance score, and notes for the final review.

Do not make all summaries equally short. Scale detail by read status and relevance:

- `metadata only`: citation plus brief reason for inclusion or exclusion.
- `abstract read`: enough detail to explain the claim, method, and likely relevance.
- `partial read`: detailed notes on the sections actually read, including evidence and limitations.
- `full read` or deep-read paper: detailed notes that are usually sufficient for later synthesis without reopening the paper.
- Math-heavy or algorithm-heavy papers: do not omit the mathematical and algorithmic explanation when it affects the answer, and write the key equations into the summary with explanation.

Do not overwrite an existing summary casually. If it exists, update it by preserving useful prior notes and adding new evidence, with a short `Update Notes` section when appropriate.

## Review, Answer, And Proposal

Generate the review first, then prepare the answer and proposal from the review and summaries. Use:

- `references/review-template.md` for literature synthesis.
- `references/run-manifest-template.yaml` for run metadata.

The answer should be the user-facing result returned in chat by default. It should include:

- direct answer
- confidence and scope
- source context used
- literature evidence
- synthesis of mechanisms
- concrete recommendations
- links to summaries and review, plus proposal details when relevant

The proposal should connect literature to the source repository. It should be returned in chat by default and include:

- source context used
- key literature themes
- likely gaps or failure modes
- concrete hypotheses
- actionable experiments or implementation changes
- risks and validation checks
- literature add candidates

Only create a saved answer or proposal file when the user explicitly asks for persisted deliverables.

## Output Layout

Use this source-repository structure unless the user requests otherwise:

```text
<source-repo>/
  paperflow/
    <request-slug>/
      request.md
      run-manifest.yaml
      run-log.md
      literature_add_candidates.md
      summaries/
      reviews/
        review.md
```

Keep generated output files inside the source repository so the supporting research travels with the code, notes, and experiments it explains. Return the main answer and proposal in chat unless the user requests saved files.

## Continuing Prior Local Work

When a user asks to continue a prior answer in the same repository:

1. Read the source request page.
2. Check `paperflow/<request-slug>/run-manifest.yaml`.
3. Read the existing review, summaries, and run log as needed:

```text
paperflow/<request-slug>/
  run-log.md
  reviews/review.md
  summaries/
```

## Safety

- Do not add PDFs or metadata to Paperpile, Zotero, BibTeX files, or any other library automatically.
- If a useful paper has no local PDF, suggest downloading or adding it, but do not download, upload, or move it without explicit approval.
- Do not upload, delete, rename, or move Google Drive files without explicit approval.
- Do not modify source code, configs, data, or experiment files unless the user explicitly requests implementation. Writing `paperflow/<request-slug>/` support files is allowed as part of this skill, but do not create `answer.md`, `proposal.md`, `literature_master.md`, or `source_context.md` unless explicitly requested.
- Be clear when a paper was found but not fully read.
- Use exact citations, DOI, arXiv ID, or URLs whenever available.
