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
- Use free web sources for missing papers.
- If a useful paper has no local PDF, mark it as missing and suggest downloading or adding the PDF.
- If equations, definitions, or algorithms matter, check them against the paper and include the verified details in the per-paper summary.
- Do not modify source code, configs, data, or experiment files unless explicitly requested.
- Do not add PDFs or metadata to Paperpile, Zotero, BibTeX files, or another library without approval.
- Be explicit about which source files were read.
