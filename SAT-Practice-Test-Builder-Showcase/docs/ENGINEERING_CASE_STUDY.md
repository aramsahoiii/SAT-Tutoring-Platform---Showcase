# Engineering case study

## From teacher intent to an enforceable test plan

The central design challenge was allowing teachers to describe a test naturally
without letting a probabilistic model control exact assessment composition.

The workflow is divided into two stages:

1. Interpret the teacher's request into a structured, editable blueprint.
2. Apply deterministic validation and selection rules to the approved blueprint.

This division keeps the interaction flexible while making the final result
explainable and testable.

## Important invariants

The application treats the following as enforceable rules:

- The final question count must equal the approved total.
- Domain and skill allocations must match the approved blueprint.
- A generated test cannot contain duplicate questions.
- Previously assigned questions can be excluded for an individual student.
- A shortage must be reported rather than filled with unrelated content.
- The same seed and eligible question set must reproduce the same selection.
- Student-facing responses must not contain answer-key information.

## Persistence model

Local development uses a SQLite working cache. Hosted deployments use Supabase
as the durable source of truth for question metadata, students, assignments,
responses, and imported results. The server hydrates its working state from the
cloud store and synchronizes workflow changes back to it.

This design supports fast deterministic selection locally while retaining a
durable hosted record.

## Import and quality pipeline

Question imports are treated as a validation pipeline rather than a simple file
upload. Imported records and visual assets are checked before they become part
of the usable question bank. Routine records can proceed automatically, while
exceptions are surfaced for focused human review.

The same principle is used for assessment-result imports: parse the external
format, normalize it into the application model, validate expected fields, and
only then update student analytics.

## Security and privacy boundaries

- Secrets are supplied through environment configuration and are not committed.
- Local databases and imported question assets remain outside version control.
- Student pages are scoped to student-safe information.
- Teacher workflows require separate protection in hosted environments.
- Public portfolio material contains no real student or licensed question data.

## AI-assisted engineering

OpenAI Codex and GitHub Copilot in Visual Studio Code were used to accelerate
development and support iteration. Their output was treated as a proposal, not
as an authority. Architectural choices, privacy requirements, and correctness
criteria remained human-owned, and proposed changes were checked against tests
and real application behavior.

This workflow was especially useful for exploring implementation alternatives,
expanding test coverage, diagnosing failures, refactoring repeated logic, and
improving project documentation.
