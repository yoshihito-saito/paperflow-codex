# paperflow-codex

`paperflow-codex` is the source repository for the `paperflow` Codex skill.

Paperflow is a literature-grounded research workflow for Codex. It helps answer questions about a codebase, experiment, model, or design decision by reading the source context, checking relevant papers, writing per-paper notes, and then producing a review and concrete next-step proposals.

Use it when you want Codex to answer questions like:

- What does the literature say about this approach?
- Why might this experiment or model be behaving this way?
- Which papers should we read before changing the implementation?
- What experiments or implementation changes should come next?

The key idea is that Codex should not jump straight from search results to a final answer. Paperflow keeps the request, paper summaries, review, and proposals beside the source project so the reasoning can be checked and updated later.

## Use As A Codex Skill

In the source repository, first ask Codex to create a request file:

```text
Read this repository and write a Paperflow Request for <question>. Save it as paperflow/<request-slug>/request.md.
```

Then run Paperflow:

```text
Use paperflow to answer paperflow/<request-slug>/request.md.
```

The request and outputs live in the source repository, not in this repository:

```text
<source-repo>/
  paperflow/
    <request-slug>/
      request.md
      run-log.md
      literature_master.md
      literature_add_candidates.md
      summaries/
      reviews/
        review.md
      proposals/
      run-manifest.yaml
```

`paperflow-codex` only stores the skill instructions and templates.

## Paperpile / Google Drive

Paperpile is the default library setup. If it is available, Paperflow should check it before open-web sources and should read matching Paperpile PDFs before web PDFs or metadata.

To make Paperpile available to Paperflow:

1. Open Paperpile settings.
2. Go to `Workflow and integrations`.
3. Configure `BibTeX exports`.
4. Save the exported bibliography as:

```text
Google Drive/paperpile.bib
```

When Codex can access that file, Paperflow uses it as the shared bibliography source across projects. If a useful paper is missing from Paperpile or has no local PDF, Paperflow records that in `literature_add_candidates.md`.

When starting a Paperflow run, you can tell Codex:

```text
Use paperflow to answer paperflow/<request-slug>/request.md. Use Google Drive/paperpile.bib as the bibliography source.
```

## Optional: Zotero Or Mendeley

Paperflow can also use Zotero or Mendeley if you provide a BibTeX file and, when available, the folder that contains PDFs.

Suggested setup:

- Zotero: export or auto-export BibTeX to `Google Drive/zotero.bib`.
- Mendeley: export BibTeX to `Google Drive/mendeley.bib`.
- PDFs: if your PDFs are stored in a normal folder, tell Codex the folder path.

For Zotero, Better BibTeX auto-export is the easiest way to keep `zotero.bib` updated. For Mendeley, export BibTeX from the app whenever the library changes.

Ask Codex to edit the skill configuration/instructions for your setup:

```text
Edit the paperflow Codex skill so it uses Zotero by default. Use Google Drive/zotero.bib as the bibliography source and /path/to/PDFs as the PDF folder.
```

or:

```text
Edit the paperflow Codex skill so it uses Mendeley by default. Use Google Drive/mendeley.bib as the bibliography source and /path/to/PDFs as the PDF folder.
```

## ChatGPT Workflow

You can also split the workflow between Codex and ChatGPT.

Use Codex to inspect the source repository and write only the request:

```text
Read this repository and write a Paperflow Request for <question>. Save it as paperflow/<request-slug>/request.md.
```

Then give ChatGPT the target repository and ask it to follow:

```text
chatgpt-workflow/PAPERFLOW_CHATGPT.md
```

ChatGPT should treat `paperflow/<request-slug>/request.md` as the contract, then use the prompt sequence in `chatgpt-workflow/prompts/` to write the literature scan, per-paper summaries, review, proposal, run log, and manifest.

Useful starting prompt for ChatGPT:

```text
Follow chatgpt-workflow/PAPERFLOW_CHATGPT.md for paperflow/<request-slug>/request.md. Create or update the Paperflow Markdown outputs in paperflow/<request-slug>/.
```

## What Gets Written

- `summaries/`: one Markdown summary per paper that was actually read.
- `reviews/review.md`: the literature review built from those summaries.
- `proposals/`: concrete next experiments or implementation changes.
- `literature_master.md`: papers considered during the run.
- `literature_add_candidates.md`: papers worth adding to Paperpile or another library.
- `run-log.md`: what happened during the run.
- `run-manifest.yaml`: paths and status for continuing later.

## Installation

For Codex to auto-discover the skill, copy or symlink:

```text
<paperflow-codex-path>/skills/paperflow
```

into:

```text
~/.codex/skills/paperflow
```

The templates used by the skill are in:

```text
skills/paperflow/references/
```
