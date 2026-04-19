# paperflow-codex

`paperflow-codex` is a lightweight Codex skill repository for literature-grounded answers to research and implementation questions.

It is designed to bridge a source request page with a Paperflow-style literature pipeline:

1. read a source request page,
2. extract only the source context needed to answer it,
3. check the user's Google Drive Paperpile library first,
4. search free web sources for missing literature,
5. create persistent per-paper summary files,
6. synthesize a scoped literature review,
7. return a direct answer, and
8. propose concrete research or implementation next steps.

The guiding idea is that an answer should be compiled from saved notes and summary files, not generated directly from a single ad hoc search.

## One-Line Use

For most requests, this is enough:

```text
Use paperflow-research to answer /path/to/source/request.md.
```

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

## Request-Response Workflow

The preferred input is a source request page inside the source repository. It should state the question, source context, files to read, literature scope, desired answer, and constraints.

```text
Use paperflow-research to answer /path/to/source/repo/paperflow-requests/<request>.md.
```

By default, results are saved in the source repository, next to the work they explain:

```text
<source-repo>/
  paperflow-requests/
    <request-slug>.md
  paperflow/
    <request-slug>/
      answer.md
      run-log.md
      literature_master.md
      paperpile_add_candidates.md
      summaries/
      reviews/
        review.md
      proposals/
        YYYY-MM-DD-<topic>.md
      run-manifest.yaml
```

This keeps the question, the answer, and the supporting notes in the same repository. `paperflow-codex` itself stays small: it only contains the skill and templates.

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
