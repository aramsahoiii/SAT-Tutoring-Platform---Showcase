# Engineering case study

## The problem and approach

Creating individualized SAT practice involves more than finding similar questions. Teachers need an exact mix of topics and difficulty, must avoid repeat assignments, and need a way to turn assessment results into useful follow-up work. The application combines an editable test plan, a validated question bank, retrieval, deterministic selection, assignment, and reporting in one workflow.

## Architecture and responsibilities

The [architecture diagram](images/system-architecture.png) shows the major components and their relationships. It is a conceptual overview: browser operations pass through the HTTP API, and the diagram's arrows summarize responsibilities rather than literal direct calls.

| Component | Responsibility |
| --- | --- |
| `static/app.js` and teacher pages | Test-building, question import and review, student records, assignment, and reporting workflows |
| `static/student.js` and student pages | Student-scoped assignments, test-taking, and performance views |
| `sat_app/server.py` | HTTP routes, static serving, and access checks |
| `sat_app/core.py` | Question bank, request parsing, blueprint and selection rules, assignments, responses, and SQLite persistence |
| `sat_app/retrieval.py` | Candidate ranking, optional embeddings, and optional grounded summaries |
| `sat_app/importers.py` | Question PDF and Bluebook workbook imports and student analytics |
| `sat_app/audit_bank.py` | Checks of question records and visual assets |
| `sat_app/supabase_backend.py` and `supabase_migrate.py` | Cloud synchronization and migration of local records |
| `api/index.py` | Vercel serverless entry point |

## Test generation and retrieval

The teacher's request is parsed into a structured blueprint that can be inspected and edited. Domain, skill, difficulty, and prior-assignment constraints determine eligibility. For topical requests, optional ranking uses BM25 lexical relevance, normalized terms, SAT concept matches, and metadata; semantic embeddings add a signal when a local or compatible service is configured. If embeddings are unavailable, the built-in ranking path remains available.

The selection engine applies the approved allocations and uses a seed for repeatability with the same eligible pool and inputs. It prevents duplicate questions and reports shortages when constraints cannot be satisfied. When a topical request has domain allocations, those domains also bound the unallocated remainder. Ranking can influence which eligible questions are selected, but cannot expand eligibility or change the approved counts.

A separately configured language model can produce a teacher-facing summary grounded in retrieved IDs, prompts, and metadata. Correct answers and rationales are excluded from that context. This optional explanation does not control selection.

## Content and student data

CSV question files are indexed into the bank. PDF imports archive the source and extract question records and student-facing visual blocks; structurally valid records can proceed while exceptions are surfaced for review. The audit checks active records for missing answers, malformed metadata, review exceptions, and visual problems, including missing, corrupt, or blank images and crop/leakage checks.

Bluebook spreadsheets are parsed into student records and analytics. The teacher can inspect test and domain results and assign a generated draft. Assignment records make questions count as previously given before completion. The student portal lists assignments and accepts responses; the student-facing test omits answer-key data. Teacher views provide separate test review and answer information.

## Persistence and access

Without Supabase configuration, the application uses a local SQLite database. With Supabase configured, it hydrates a SQLite working cache from the cloud and synchronizes workflow changes back to Supabase, which acts as the durable store. Hosted configuration includes a teacher access password and Supabase-backed student authentication; local student pages are separated by workflow but do not themselves provide production-grade identity enforcement. Vercel configuration supports a private hosted deployment with a temporary cache on cold starts.

## Verification and scope

The reviewed private snapshot's Python suite ran 102 tests successfully using the project's virtual environment. It covers selection and shortages, reproducibility, retrieval, imports, student and teacher routes, and Supabase synchronization and migration. The project also contains frontend tests and a separate whole-bank audit; the passing Python suite alone does not establish production readiness or validate every deployment and dataset.

The code and data are private. This case study documents the inspected implementation rather than claiming access to a public live demo, production usage, or measured learning outcomes. OpenAI Codex and GitHub Copilot assisted development; the developer retained responsibility for requirements, review, and technical decisions.
