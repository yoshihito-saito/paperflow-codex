# Paperflow Request: <short title>

## Question

State the question that paperflow should answer.

## Source

- Source repository:
- Source page:
- Request slug:

## Source Context

Summarize the relevant project context, observed behavior, assumptions, and claim boundaries. This section should be enough to guide the literature search.

## Source Files To Read

- `README.md`
- `src/...`
- `documents/...`
- `configs/...`

## Literature Scope

- topic 1
- topic 2
- topic 3

## Literature Search Terms

- search term 1
- search term 2
- search term 3

## Research Depth

- Depth preset: standard
- Candidate scan: up to 40 papers
- Per-paper summaries: up to 20 papers
- Deep reads: up to 10 papers
- Reading mode: one paper at a time; write or update its summary before opening the next paper
- Summary detail: detailed per-paper notes by default; use compact synthesis only inside each summary as an entry point for review writing, never as the only summary artifact

## Local Library Sources

- Paperpile / Google Drive folder:
- Canonical Paperpile bibliography path: `Google Drive/Paperpile/paperpile.bib`
- Local bibliography files:
- Existing notes or literature lists:

## Desired Answer

- direct answer
- relevant paper summaries
- scoped literature review
- mechanism comparison
- checked equations, definitions, or algorithms when they matter
- proposal or next actions

## Output

- Default output folder: `paperflow/<request-slug>/`
- Request file: `paperflow/<request-slug>/request.md`

## Constraints

- Use existing paper-library sources before web search when available.
- If Paperpile / Google Drive is available, search it first.
- If Paperpile has a matching PDF for a candidate paper, read that PDF before using an open web PDF, publisher page, abstract, or metadata.
- When web search finds a paper, check Paperpile for a matching PDF before reading or summarizing the web source.
- Use the canonical shared Paperpile `.bib` path when configured. By default this should be `Google Drive/Paperpile/paperpile.bib`.
- If the canonical `.bib` file is missing and library access is available, create it once there.
- If the canonical `.bib` file exists and library access is available, update only changed entries instead of regenerating the whole file.
- If the canonical `.bib` file cannot be found or accessed in the session, show a clear message that the Paperpile bibliography could not be found and record the issue in the run log.
- Use free web sources for missing papers.
- If a useful paper has no local PDF, mark it as missing and suggest downloading or adding the PDF.
- If equations, definitions, or algorithms matter, check them against the paper and include the verified details in the per-paper summary.
- Do not modify source code, configs, data, or experiment files unless explicitly requested.
- Do not modify Paperpile, Zotero, or another upstream library without approval. Updating the canonical shared `.bib` file is allowed.
- Be explicit about which source files were read.
