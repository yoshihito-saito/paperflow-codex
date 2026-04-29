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
10. Prioritize papers for reading depth before summarizing: core papers, supporting papers, background papers, and exclusions.
11. Process papers one at a time. After reading each paper beyond metadata level, immediately create or update its Markdown summary before opening the next paper.
12. For every core paper, write section-by-section notes before using it as evidence in the review. Do not treat a short abstract-style note as a completed summary for a core paper.
13. Run the per-paper summary quality check from `references/summary-template.md`; mark incomplete summaries clearly and do not rely on them for strong claims.
14. Synthesize a scoped review for the request from the paper summaries. Major review claims should trace back to summary files, not to uncaptured memory of papers.
15. Run the review output compliance check against the requested output and quality constraints in `request.md`.
16. Produce an answer that directly responds to the request, with links to the supporting files, and return it in chat unless the user explicitly asks for a saved answer file.
17. Produce a proposal with concrete experiments, implementation changes, or decision points when the request asks for next steps, and return it in chat unless the user explicitly asks for a saved proposal file.
18. Do not create standalone `source_context.md` or `literature_master.md` files unless the user explicitly requests persisted copies.
19. Update `run-manifest.yaml` and `run-log.md`.
20. Suggest papers to add to the user's library, but do not add, upload, move, rename, or delete files without explicit user approval.

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

Use `standard` depth by default unless the request specifies otherwise. Depth controls both the number of papers screened and the thickness of notes for high-priority papers; `deep` must not mean many shallow summaries.

- `quick`: scan up to 20 candidate papers, summarize up to 5 papers, deeply read up to 2 papers.
- `standard`: scan up to 40 candidate papers, summarize up to 20 papers, deeply read up to 10 papers. Core papers should receive at least `section-level read` notes when their full text is available.
- `deep`: scan 50 or more candidate papers when useful, summarize up to 25 papers, deeply read up to 15 papers. Core papers must receive `deep read with section notes` when their full text is available; high-value supporting papers should receive at least `section-level read` notes.

Use these read-status labels consistently:

- `metadata only`: citation or index record only; do not use for substantive claims.
- `abstract only`: abstract and metadata read; use only for tentative inclusion, exclusion, or gap notes.
- `skimmed full text`: full text opened and selectively inspected; use only for low-stakes support unless source locations are recorded.
- `section-level read`: relevant paper sections read and summarized separately with source locations.
- `deep read with section notes`: abstract, introduction, methods, results, discussion, important figures/tables, and relevant supplement checked and summarized section by section.

Before reading, assign each candidate a priority:

- `core`: central to the answer, proposal, or evidence table.
- `supporting`: useful but not load-bearing.
- `background`: framing, methods, or historical context.
- `exclude`: screened out with a reason.

Core papers require section-by-section summaries before review synthesis. If full text is unavailable, explicitly downgrade the evidence strength and list the paper in `literature_add_candidates.md`.

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
- For review, answer, and proposal synthesis, read the compact synthesis subsection inside each per-paper summary first and then open the detailed sections for high-relevance papers.
- Re-open original papers only when a claim needs verification or the detailed summary is insufficient.
- Avoid copying long passages from papers, but preserve detailed explanations, equations, methods, evidence, limitations, and source-repository relevance in your own words.
- When PDF-to-Markdown or JSON extraction is used, do not pass raw extracted output directly into the model context.
- Clean boilerplate, references, repeated headers or footers, and OCR artifacts first when possible.
- Then process the cleaned paper section by section.
- Use extracted JSON primarily for source locations, equations, figures, and tables when needed.
- Use the cheapest capable model by default for metadata extraction, PDF or Markdown cleanup, section-level notes, ordinary empirical paper summaries, and citation normalization.
- Use a stronger reasoning model for core papers or sections that are math-heavy, theory-heavy, proof-heavy, algorithmically central, marked `needs verification`, or used as primary evidence in the final review.
- For theoretical papers that are core to the answer, use a stronger reasoning model for the model setup, assumptions, definitions, key equations, derivations, propositions, proofs, and request-specific interpretation.
- Do not use a stronger reasoning model for raw PDF reading by default.
- Use stronger reasoning for targeted verification, mathematical or algorithmic sections, theoretical sections, proof checks, and final synthesis when needed.

## Mathematical And Algorithmic Accuracy

When a paper's contribution depends on equations, definitions, objectives, update rules, architectures, or algorithmic assumptions, verify those details from the paper before adding them to the summary.

- Preserve important equations in concise LaTeX form when they are needed for the review or proposal.
- Write important equations directly into the per-paper summary when they matter for the request, not just a prose reference to them. For core theoretical papers, include the central model equations even when the final answer will only use the intuition.
- Explain each important equation in prose, including symbol meanings, dimensions, assumptions, loss terms, constraints, boundary conditions, and equation numbers when available.
- Include term-by-term intuition and why the equation matters for the source request.
- Distinguish exact formulas from paraphrased intuition.
- For derivations, summarize the logical dependency: which definitions, assumptions, lemmas, approximations, or optimization steps lead to the result.
- For propositions, theorems, or proofs, record the statement, proof strategy, key assumptions, and where the result is used in the paper. Mark proof steps that were not checked as `needs verification`.
- If an equation, derivation step, or notation is uncertain, mark it as `needs verification` instead of guessing.
- For algorithms, capture the inputs, outputs, core steps, and any stated complexity or convergence conditions when relevant.

## Per-Paper Summaries

Create or update one Markdown file for every paper that is read beyond metadata level. Use:

`paperflow/<request-slug>/summaries/<year>-<first-author>-<short-title>.md`

Use `references/summary-template.md` as the template. Each summary must preserve citation metadata, source links, library status, PDF status, read status, evidence strength, one-sentence takeaway, section-by-section notes when required, methods, key findings, mathematical or algorithmic details when relevant, limitations, extrapolation boundaries, relationship to other papers, relevance score, and notes for the final review.

The summary is a durable paper card, not an abstract. For core papers, it should explain what the paper actually did section by section: question, apparatus or dataset, subjects, task structure, manipulations, measurements, analyses, main results, key figures/tables, limitations, and request-specific implications.

Do not replace per-paper summaries with a single compact digest file. Files such as `core-literature-compact.md` may be created only as optional navigation aids after the per-paper summaries exist, never instead of them.

Do not make all summaries equally short. Scale detail by read status and relevance:

- `metadata only`: citation plus brief reason for inclusion or exclusion.
- `abstract only`: enough detail to explain the claim, method, and likely relevance, but mark evidence as tentative.
- `skimmed full text`: identify which sections and figures were inspected and why the paper is not core.
- `section-level read`: detailed notes on each section actually read, including source locations, evidence, and limitations.
- `deep read with section notes`: detailed notes that are usually sufficient for later synthesis without reopening the paper.
- Math-heavy, theory-heavy, proof-heavy, or algorithm-heavy papers: do not omit the mathematical, theoretical, proof, or algorithmic explanation when it affects the answer. Write the key equations, assumptions, definitions, derivation dependencies, propositions, and algorithm steps into the summary with explanation.

Minimum quality gate:

- A core-paper summary may not be only a citation plus a few bullets.
- A summary that lacks methods, results, limitations, and request-specific relevance should be marked incomplete.
- A paper cannot support a strong review claim unless the summary records the relevant evidence and source location.
- For empirical papers used in proposals, the summary must capture task structure, manipulations, measurements, and analysis logic.
- For theoretical papers used in the review, the summary must capture the model setup, assumptions, key equations, what each term means, central result, proof or derivation sketch, limitations, and request-specific interpretation.
- For requests asking for proposals, decisions, predictions, or experiment design, summaries must state which request-specific variables, controls, assumptions, evaluation criteria, or expected outcomes the paper informs.
- A run that produces only a single combined summary file, or that omits `reviews/review.md` when a review was requested, is incomplete.

Do not overwrite an existing summary casually. If it exists, update it by preserving useful prior notes and adding new evidence, with a short `Update Notes` section when appropriate.

## Review, Answer, And Proposal

Generate the review first, then prepare the answer and proposal from the review and summaries. Use:

- `references/review-template.md` for literature synthesis.
- `references/run-manifest-template.yaml` for run metadata.

The review must be traceable to summaries. Include an evidence table when the request asks for a report, recommendation, proposal, prediction, or decision. Distinguish direct evidence from extrapolation along the dimensions that matter for the request, such as species, setting, population, dataset, task, method, measurement, intervention, implementation context, or analysis method.

Before finalizing the review, compare it against `request.md`:

- every requested output section is present or explicitly marked out of scope
- every requested topic is addressed
- concrete request-specific deliverables are included when requested
- predictions, decisions, recommendations, or next steps are organized by the request's relevant units when requested
- practical recommendations include controls, risks, and validation checks when requested
- major claims link back to per-paper summaries
- direct evidence, close analogs, theory, and speculation are separated

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
