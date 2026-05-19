# 01 Read Request

You are running RefFlow through ChatGPT. Read `refflow/<request-slug>/request.md` first and treat it as the only contract.

## Tasks

- Extract the central question.
- Extract source files to read.
- Extract literature scope and search terms.
- Extract constraints and desired output.
- Identify any missing information or ambiguity.
- Create or update `run-manifest.yaml`, `run-log.md`, `literature_master.md`, and `literature_add_candidates.md` if they do not exist.

## Rules

- Do not create paper summaries yet.
- Do not write the review yet.
- Do not scan the whole project repository.
- Read only source files named in `request.md`, unless the request is too underspecified to proceed.
- Do not modify source code, configs, data, or experiment files.
- Record missing information in `run-log.md`.

## Output

Commit the created or updated RefFlow control files to GitHub when finished.
