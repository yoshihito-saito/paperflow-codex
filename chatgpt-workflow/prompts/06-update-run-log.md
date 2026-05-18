# 06 Update Run Log

Update the run tracking files after a ChatGPT Paperflow session.

## Record In `run-log.md`

- Source files read this session.
- Papers investigated this session.
- Papers summarized this session.
- Files created or updated.
- Missing evidence that remains unresolved.
- PDF or local availability issues.
- Next action for the next session.

## Update In `run-manifest.yaml`

- Status fields.
- Completed summaries.
- Pending papers.
- Missing evidence.
- Next action.
- Last updated date.

## Rules

- Keep the log factual and concise.
- Do not rewrite history; append a dated entry.
- Do not claim a paper was read unless a summary exists or the read status is explicitly recorded.

## Output

Commit `run-log.md` and `run-manifest.yaml` to GitHub.
