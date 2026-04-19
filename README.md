# paperflow-codex

`paperflow-codex` is a lightweight Codex workflow repository for literature-grounded answers to research and implementation questions.

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
Use paperflow-research to answer /path/to/source/request.md and save the results in <paperflow-codex-path>.
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
        project-manifest-template.yaml
  requests/
    example-navigation-request.md
  projects/
    INDEX.md
    <project-slug>/
      manifest.yaml
      research-request.md       # legacy single-request entrypoint
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
      paperpile_add_candidates.md
```

## Request-Response Workflow

The preferred input is a source request page. The page can live in a source repository, a notes folder, or this repository. It should state the question, source context, files to read, literature scope, desired answer, and constraints.

```text
Use the paperflow-research skill.
Answer the request in /path/to/source/repo/documents/PAPERFLOW_REQUESTS/<request>.md.
Save the results in <paperflow-codex-path>.
```

The normalized request and answer live in this repository:

```text
projects/<project-slug>/requests/<request-slug>.md
projects/<project-slug>/answers/<request-slug>.md
```

The supporting output files also live in this repository:

```text
projects/<project-slug>/summaries/
projects/<project-slug>/reviews/<request-slug>-review.md
projects/<project-slug>/proposals/YYYY-MM-DD-<topic>.md
projects/<project-slug>/literature_master.md
```

Each project should also have a `manifest.yaml` that records source repositories, latest request and answer files, output paths, and related projects. This makes the answer package callable later from another repository or another Codex session.

Start from:

```text
skills/paperflow-research/references/project-manifest-template.yaml
skills/paperflow-research/references/source-request-template.md
skills/paperflow-research/references/answer-template.md
```

## Reusing Answers From Other Repositories

From another repository or source page, ask Codex to use the manifest or index:

```text
Use the paperflow-research skill.
Load <paperflow-codex-path>/projects/INDEX.md,
then reuse the saved answers for <project-slug>.
Compare them with the current request and propose what carries over.
```

When a new repository is related to an existing project, add it under `related_source_repositories` in that project's `manifest.yaml`, or create a new project folder and link the prior project under `related_projects`.

## Basic Use

Ask Codex to use the `paperflow-research` skill with a source request:

```text
Use paperflow-research to answer projects/<project-slug>/requests/<request-slug>.md and save the results in <paperflow-codex-path>.
```

For actual work, copy or adapt `skills/paperflow-research/references/source-request-template.md` into `projects/<project-slug>/requests/<request-slug>.md` or into a source repository page that Codex can read.

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
