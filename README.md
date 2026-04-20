# paperflow-codex

`paperflow-codex` is a Codex skill for answering research and implementation questions with literature support.

Use it when you have a source repository and want Codex to answer a question such as:

- Why is this model or experiment behaving this way?
- What does the relevant literature say about this approach?
- What papers should be summarized before making a design decision?
- What experiments or implementation changes should come next?

The skill reads a request file, checks your existing paper library if one is available, searches the open web for missing papers, writes one summary per important paper, then writes an answer, review, and proposal. Paperpile / Google Drive is optional, but if it is available the skill should search it first.

The important idea is simple: do not jump straight from search results to a final answer. Save the paper summaries and notes that support the answer, so the reasoning can be checked and updated later.

## One-Line Use

For most requests, this is enough:

```text
Use paperflow-research to answer /path/to/source/request.md.
```

The request file should usually live in the source repository:

```text
<source-repo>/paperflow-requests/<request-slug>.md
```

The results are written back into that same source repository:

```text
<source-repo>/paperflow/<request-slug>/
```

`paperflow-codex` itself only stores the skill instructions and templates.

## Layout

```text
paperflow-codex/
  skills/
    paperflow-research/
      SKILL.md
      references/
        research-request-template.md
        source-request-template.md
        answer-template.md
        summary-template.md
        review-template.md
        proposal-template.md
        run-manifest-template.yaml
  requests/
    example-navigation-request.md
```

## How It Works

A request file is a Markdown file that tells Codex what question to answer and what source context matters. It can be short. For example:

```text
# Paperflow Request: Unexpected Benchmark Regression

## Question

Why did the latest implementation improve one benchmark but regress another, and what does the literature suggest we should check next?

## Source Files To Read

- README.md
- src/
- experiments/
- docs/benchmark-notes.md

## Literature Scope

- methods related to the implementation change
- evaluation metrics used by the benchmark
- known failure modes or tradeoffs
```

Then ask Codex:

```text
Use paperflow-research to answer paperflow-requests/benchmark-regression.md.
```

By default, the skill saves results in the source repository:

```text
<source-repo>/
  paperflow-requests/
    <request-slug>.md
  paperflow/
    <request-slug>/
      answer.md
      run-log.md
      literature_master.md
      literature_add_candidates.md
      summaries/
      reviews/
        review.md
      proposals/
        YYYY-MM-DD-<topic>.md
      run-manifest.yaml
```

This keeps the question, the answer, and the supporting notes beside the code or experiment they explain.

## What Gets Written

- `answer.md`: the direct answer to the request.
- `summaries/`: one Markdown summary per paper that was actually read.
- `reviews/review.md`: the literature review built from the summaries.
- `proposals/`: concrete next experiments or implementation changes.
- `literature_master.md`: a table and notes that track all papers considered.
- `literature_add_candidates.md`: papers that may be worth adding to Paperpile, Zotero, a `.bib` file, or another local library. If a useful paper has no local PDF, the skill should say so here and suggest downloading or adding the PDF.
- `run-log.md`: what was done in this run.
- `run-manifest.yaml`: status and paths for continuing the run later.

If you later want to add new keywords or papers, use the same folder:

```text
Use paperflow-research to update paperflow/<request-slug> with keywords "new keyword, another keyword".
```

Start from:

```text
skills/paperflow-research/references/source-request-template.md
skills/paperflow-research/references/answer-template.md
skills/paperflow-research/references/run-manifest-template.yaml
```

## Basic Use

Ask Codex to use the `paperflow-research` skill with a source request:

```text
Use paperflow-research to answer paperflow-requests/<request-slug>.md.
```

For actual work, copy or adapt `skills/paperflow-research/references/source-request-template.md` into the source repository, usually under `paperflow-requests/<request-slug>.md`.

## Installation As A Codex Skill

For Codex to auto-discover the skill, copy or symlink:

```text
<paperflow-codex-path>/skills/paperflow-research
```

into:

```text
~/.codex/skills/paperflow-research
```

This repository keeps the skill source-controlled, while the symlink or copy makes it available to Codex.
