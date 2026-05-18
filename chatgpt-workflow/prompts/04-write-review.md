# 04 Write Review

Write `paperflow/<request-slug>/reviews/review.md` using only existing files in `paperflow/<request-slug>/summaries/` as literature evidence.

## Tasks

- Read `request.md`.
- Read all relevant summaries.
- Structure the review around the deliverables requested in `request.md`.
- Include a direct answer.
- Include evidence by requested topic.
- Include an evidence table.
- For every major claim, cite the corresponding summary path.
- Mark unsupported claims as `missing evidence`.
- Distinguish direct evidence, close analog, methodological support, theory/background, and speculation.
- Update `run-manifest.yaml` and `run-log.md`.

## Rules

- Do not cite papers that do not have summaries.
- Do not rely on uncaptured memory of papers.
- Reopen original papers only when a summary is insufficient or a claim needs verification; if reopened, update the summary first.
- Preserve uncertainty and limitations.

## Output

Commit `reviews/review.md` and updated tracking files to GitHub.
