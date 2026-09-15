# Project Instructions

## Goal

Build a small, maintainable MVP for automated discovery and qualification
of Russian B2B companies as potential customers for AI automation services.

The pipeline will eventually:

1. ingest official company data;
2. store raw source data;
3. normalize data;
4. filter companies;
5. enrich filtered companies;
6. calculate lead scores;
7. export qualified leads.

## Development principles

- Keep the implementation simple.
- Do not introduce microservices unless clearly necessary.
- Prefer Python standard library and already selected dependencies.
- Do not add infrastructure without a concrete MVP requirement.
- Every external source must be verified before implementation.
- Never invent API endpoints, fields, formats, authentication methods,
  rate limits or update schedules.
- Prefer official documentation and official source data.
- If something cannot be verified, explicitly mark it as unverified
  and do not build the implementation around it.
- Preserve raw source data.
- Normalized data must be traceable back to its source.
- Make ingestion idempotent.
- Handle network errors, timeouts and retries where appropriate.
- Log enough information to debug failures.
- Do not hide errors with broad exception handling.
- Do not over-engineer error handling.

## Code style

- Python.
- Small functions with one clear responsibility.
- Type hints for public functions.
- Docstrings for non-trivial functions.
- Comments only where they explain WHY, not obvious WHAT.
- Avoid unnecessary abstractions.
- Avoid premature generic frameworks.
- Prefer straightforward code over clever code.

## Testing

Every implementation step must include tests where practical.

Tests should cover:
- normal behavior;
- invalid input;
- important edge cases;
- idempotency where applicable.

Run tests after every step.

Do not proceed to the next implementation step if the current step
does not pass its validation criteria.

## Documentation

After every completed step:

1. update README.md if user-facing behavior changed;
2. document how to run the implemented component;
3. document input/output;
4. document important assumptions;
5. document verified source information;
6. document known limitations.

## Git

Make one small commit per completed implementation step.

Commit messages should be simple, for example:

step 1: add RSMP downloader

Do not combine unrelated steps into one commit.

## Important

Do NOT implement the entire project at once.

Work only on the current step from IMPLEMENTATION_PLAN.md.

Before modifying code:

1. read AGENTS.md;
2. read IMPLEMENTATION_PLAN.md;
3. inspect the existing project;
4. identify the current step;
5. implement only that step.

After implementation:

1. run tests;
2. run relevant validation;
3. update documentation;
4. show changed files;
5. summarize what was implemented;
6. state whether the step is complete;
7. STOP.

Never automatically continue to the next step.