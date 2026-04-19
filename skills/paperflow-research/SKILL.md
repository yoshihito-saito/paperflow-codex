---
name: paperflow-research
description: Use when Codex is asked to answer a source request page with literature-grounded research, check the user's Google Drive Paperpile library first, search free web sources for missing papers, create per-paper summaries, synthesize a scoped review, and return an answer plus concrete proposals.
---

# Paperflow Research

## Core Principle

Treat literature-grounded answering as a reproducible request-response pipeline, not a one-shot answer. The main input is a source request page: a Markdown page from a source repository, notes folder, or user-provided path that states a question, source context, relevant files, and desired output.

Preserve intermediate files: copied/normalized request, per-paper summaries, literature master, scoped review, answer, proposal, run log, and Paperpile add candidates.

Use the request page as authoritative. Read only the source files named in the request unless the request is underspecified or the user asks for broader inspection. Source context extraction supports the answer; it is not the primary goal.

Store generated research files in `paperflow-codex/projects/<project-slug>/` by default, not in the source repository. Treat each project folder as a reusable knowledge package that can be loaded later from other repositories.

## One-Line Invocation

When the user says something like this, run the full workflow:

```text
Use paperflow-research to answer /path/to/source/request.md and save the results in <paperflow-codex-path>.
```

Interpret "save the results" as creating or updating the project folder under `projects/<project-slug>/` with the request, answer, summaries, review, proposal, run log, and Paperpile add candidates.

## Standard Workflow

1. Read the source request page, or create one from `references/source-request-template.md` if the user gives the request in chat.
2. Extract the project slug, source repository or source page path, research question, source context, named source files, literature scope, Paperpile search terms, and desired answer format.
3. Create or update the matching `paperflow-codex/projects/<project-slug>/` folder and preserve the normalized request under `requests/`.
4. Create or update `manifest.yaml`, `run-log.md`, `literature_master.md`, and `paperpile_add_candidates.md`.
5. Read only the source files named in the request. If no files are named and the source context is insufficient, do a narrow fallback read of `README*`, cited docs, or obvious config/model files.
6. Search the user's Google Drive Paperpile folder first.
7. Reuse existing metadata, notes, summaries, and PDFs when available.
8. Deduplicate papers before web search using DOI, arXiv ID, PMID, then normalized title.
9. Search free web sources only for gaps: arXiv, PubMed, OpenAlex, Crossref, publisher pages, author pages, GitHub, and project pages.
10. Create or update one Markdown summary file per paper read beyond metadata level.
11. Build or update `literature_master.md` from the paper summaries.
12. Synthesize a scoped review for the request.
13. Produce an answer that directly responds to the request, with links to the supporting files.
14. Produce a proposal with concrete experiments, implementation changes, or decision points when the request asks for next steps.
15. Update `manifest.yaml`, `projects/INDEX.md`, and `run-log.md`.
16. Suggest open-access PDFs to add to Paperpile, but do not add or move files without explicit user approval.

## Source Request Handling

The source request page is the contract. It should contain:

- the question to answer
- the source repository or source page path
- the relevant source context
- the source files to read
- the literature scope and Paperpile search terms
- the desired output
- constraints and claim boundaries

Use `references/source-request-template.md` for new requests. If the user supplies a source page from another repository, copy or normalize it into:

`projects/<project-slug>/requests/<request-slug>.md`

Do not edit the original source page unless the user explicitly asks.

## Source Context Extraction

Source context extraction is narrow and request-guided. Start with files named in the source request, such as:

- a design note or issue page
- `README*`
- experiment notes, notebooks, or logs
- model, environment, dataset, and training code
- configuration files

If the request already contains enough context, source file reads may be minimal. If the request is underspecified, read the smallest set of source files needed to answer accurately, then record what was inspected in the answer and proposal.

## Paperpile-First Literature Search

When Google Drive tools are available:

1. Find the user's Paperpile folder or the folder specified in the request.
2. Search for existing papers using query terms from the source request and source context.
3. Include PDFs, Google Docs, Sheets, BibTeX, notes, and existing literature lists.
4. Record whether each paper is already in Paperpile.

When Drive access is unavailable or blocked, state that limitation and continue with local files and free web sources.

## Web Search Rules

Use web search after checking Paperpile. Prefer stable, free sources:

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

## Per-Paper Summaries

Create or update one Markdown file for every paper that is read beyond metadata level. Use:

`projects/<project-slug>/summaries/<year>-<first-author>-<short-title>.md`

Use `references/summary-template.md` as the template. Each summary must preserve citation metadata, source links, Paperpile status, one-sentence takeaway, methods, key findings, limitations, relationship to other papers, relevance score, and notes for the final review.

Do not overwrite an existing summary casually. If it exists, update it by preserving useful prior notes and adding new evidence, with a short `Update Notes` section when appropriate.

## Review, Answer, And Proposal

Generate the review, answer, and proposal only after the relevant per-paper summaries exist. Use:

- `references/review-template.md` for literature synthesis.
- `references/answer-template.md` for the request answer.
- `references/proposal-template.md` for project diagnosis and next steps.
- `references/project-manifest-template.yaml` for reusable project metadata.

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
- Paperpile add candidates

## Output Layout

Use this structure unless the user requests otherwise:

```text
projects/<project-slug>/
  manifest.yaml
  research-request.md              # legacy single-request entrypoint, optional
  requests/
    <request-slug>.md
  answers/
    <request-slug>.md
  literature_master.md
  run-log.md
  summaries/
  reviews/
    <request-slug>-outline.md
    <request-slug>-review.md
  proposals/
    YYYY-MM-DD-<topic>.md
  paperpile_add_candidates.md
```

Keep generated output files in `paperflow-codex`, not inside the source repository, unless the user explicitly asks to modify the source repository.

Also keep `projects/INDEX.md` updated. It should list every project slug, source repository, latest request, latest answer, review path, proposal path, and relationship to other project folders.

## Cross-Request And Cross-Repository Reuse

When a user asks from another repository or source page to reuse prior work:

1. Load `projects/INDEX.md`.
2. Identify the relevant project folder by source repository path, topic, or related project.
3. Read that project's `manifest.yaml`.
4. Read the relevant prior request and answer under `requests/` and `answers/`.
5. Read `literature_master.md`, relevant summaries, scoped reviews, and the latest proposal as needed.
6. Compare the prior answer against the new request rather than assuming conclusions transfer unchanged.
7. If the new repository becomes a separate research target, create a new `projects/<new-project-slug>/manifest.yaml` and link the prior project under `related_projects`.

## Safety

- Do not add PDFs to Paperpile automatically.
- Do not upload, delete, rename, or move Google Drive files without explicit approval.
- Do not modify the source repository unless the user explicitly requests implementation.
- Be clear when a paper was found but not fully read.
- Use exact citations, DOI, arXiv ID, or URLs whenever available.
