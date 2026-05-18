---
name: paperflow
description: Use when Codex is asked to answer a source request page with literature-grounded research, check the user's existing paper library when available, search free web sources for missing papers, create detailed per-paper summaries, synthesize a scoped review, and return a direct answer in chat plus concrete proposals.
---

# Paperflow

## Purpose

Treat literature-grounded answering as a reproducible workflow, not a one-shot answer.

The request file is the contract.
Use it to decide:

- what question is being answered
- which source files to read
- which outputs are required
- what claim boundaries and constraints apply

Write durable research artifacts into the source repository under:

`paperflow/<request-slug>/`

Use the templates in `references/` for request, summary, review, and run-manifest structure.

## Standard Workflow

1. Read or create `paperflow/<request-slug>/request.md`.
2. Create or update `run-manifest.yaml`, `run-log.md`, and `literature_add_candidates.md`.
3. Read only the source files named in the request unless the request is underspecified.
4. Check the canonical shared Paperpile bibliography path from the request or manifest. By default this should be the Google Drive `paperpile.bib` file.
5. If the canonical `.bib` file exists and is accessible, update only changed entries.
6. If the canonical `.bib` path is configured but the file is missing and library access is available, create it once there.
7. If the canonical `.bib` path cannot be found or accessed in the session, show a clear user-visible message that the Paperpile bibliography could not be found, and record the exact issue in `run-log.md` and `run-manifest.yaml`.
8. For every candidate paper, check Paperpile for a matching PDF before reading from web sources. If Paperpile has the PDF, read that PDF and record the Drive file/path in the summary.
9. Reuse existing metadata, notes, summaries, PDFs, and BibTeX entries when available.
10. Search free web sources only for gaps that remain after library checks. When web search finds a paper, check Paperpile again for a matching PDF before reading or summarizing the web copy.
11. Assign paper priority before deep reading: `core`, `supporting`, `background`, or `exclude`.
12. Read one paper at a time and write or update its summary before moving to the next paper.
13. Build the review from the per-paper summaries.
14. Return the direct answer and proposal in chat by default unless persisted files are explicitly requested.

## Library Rules

- Prefer one shared canonical `.bib` file named `paperpile.bib` at the Google Drive root.
- Zotero or Mendeley `.bib` exports may be used when the request names them as bibliography sources.
- Do not silently replace the canonical bibliography with a newly generated project-local bibliography.
- Project-local `.bib` files may be used only when they already exist as separate project materials, not as an automatic fallback.
- Update the canonical `.bib` incrementally, not by rewriting the entire file from scratch.
- Match entries by DOI first, then arXiv ID, PMID, stable library ID when available, and finally normalized title plus year.
- If an upstream deletion cannot be matched confidently, keep the local entry and record the ambiguity.
- Treat configured local library PDFs as the highest-priority full-text source. If a candidate paper has a matching Paperpile, Zotero, Mendeley, or configured local PDF, read that PDF before using an open web PDF, publisher page, abstract, or metadata.
- Check configured local PDF availability for every candidate paper found through web search, DOI lookup, arXiv, PubMed, Semantic Scholar, Google Scholar snippets, or source notes before writing the paper summary.
- Do not mark a paper as `PDF missing` or summarize it from web metadata until configured local PDF lookup has been attempted and recorded.
- Do not modify Paperpile, Zotero, or another upstream library directly.
- Updating the configured canonical shared `.bib` file is allowed.

## Summary Rules

Create one summary file per paper read beyond metadata level:

`paperflow/<request-slug>/summaries/<year>-<first-author>-<short-title>.md`

Each summary must use `references/summary-template.md`.

Each summary must include:

- a compact whole-paper overview
- explicit read status and evidence strength
- enough detail to support later review writing without guessing

Each summary must be question-conditioned:

- write the whole-paper overview for the full paper
- write detailed notes for the sections most relevant to the current request
- keep less relevant sections as brief coverage notes
- if the request depends on the paper as a whole, write detailed section-by-section notes for the whole paper

Summary depth scales by priority and read status:

- `metadata only`: citation plus inclusion or exclusion reason
- `abstract only`: tentative claim, method, and likely relevance
- `skimmed full text`: note which sections were checked and why
- `section-level read`: detailed notes for request-relevant sections with source locations
- `deep read with section notes`: detailed notes across the whole relevant paper

Core-paper rules:

- `core` papers must have section-level notes or deeper before they support major review claims
- a core-paper summary cannot be only a short bullet list
- if equations, definitions, proofs, or algorithms matter, record them in the summary and explain them
- if a detail is uncertain, mark it `needs verification`

Do not replace per-paper summaries with one combined digest file.
Optional compact digest files may exist only after the per-paper summaries exist.

## Review Rules

The review must use `references/review-template.md`.

Build the review from the summary set, not from uncaptured memory of papers.

Major review claims must be traceable to per-paper summaries.
Distinguish:

- direct evidence
- close analogs
- extrapolation
- theory or background framing
- speculation

The review must answer the requested deliverables in `request.md`.
Use the request's own categories and wording where possible.

## Efficient Reading Rules

Keep context lean while preserving fidelity:

- do not batch-read many full papers into one context
- process papers section by section when needed
- use the compact overview as the entry point, not as the whole summary
- reopen the original paper only when the saved summary is insufficient or a claim needs verification

For theoretical, mathematical, proof-heavy, or algorithmically central papers:

- use stronger reasoning for the relevant sections
- preserve the key equations or statements in the summary
- explain what the terms, assumptions, and derivation steps mean
- record proof or derivation coverage honestly

## Failure Conditions

A run is incomplete if any of the following are true:

- the canonical Paperpile bibliography could not be found and this was not reported clearly
- a candidate paper was summarized from web sources without checking whether Paperpile has a matching PDF
- per-paper summaries are missing
- a core-paper summary is only a short bullet list
- a requested review artifact is missing
- major review claims cannot be traced to summary evidence
- the run ends with only a combined digest file instead of per-paper summaries

## Safety

- Do not upload, delete, rename, or move Google Drive files without explicit approval.
- Do not add PDFs or metadata to upstream libraries automatically.
- Do not modify source code, configs, data, or experiment files unless explicitly requested.
- Do not create standalone `source_context.md` or `literature_master.md` unless explicitly requested.
- If a useful paper has no local PDF, mark that clearly and add it to `literature_add_candidates.md`.
