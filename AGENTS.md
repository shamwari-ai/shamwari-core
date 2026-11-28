# AGENTS.md — shamwari-core

Rules for AI agents and people working in this repository. They layer on the
org-wide guide, [nyuchi/.github AGENTS.md](https://github.com/nyuchi/.github/blob/main/AGENTS.md).
Where the two conflict, this file wins. Read `README.md` first.

## What this repo is

The planned home of Shamwari Core: the service that holds the database
credentials and the authoritative scope gate, in Python 3.11 and FastAPI. It
does not own the code yet. `main` holds a licence, CI wiring, lint config and
the README. The service lives in
[`shamwari-ai/shamwari`](https://github.com/shamwari-ai/shamwari) under `core/`
and `db/`, and code changes land there until the extraction into this repo is
done. The monorepo's
[`AGENTS.md`](https://github.com/shamwari-ai/shamwari/blob/main/AGENTS.md)
holds the project rules and the applied-migration log.

Once imported, Core serves `POST /auth/verify`, `POST /ground/search`,
`GET /guardrails`, `POST /sink/bulk`, `GET /rollup` and `GET /health` (README).
Key issuance, key revocation and per-key usage breakdown are missing, and
[`shamwari-platform`](https://github.com/shamwari-ai/shamwari-platform) is
blocked on them.

## The two rules

Do not weaken either without the owner's written sign-off.

1. **Personal-scope content never reaches a third-party inference provider.**
   `main.py::resolve_scope` is the authoritative half of the gate. The
   fail-fast half is `src/scope.ts` in
   [`shamwari-gateway`](https://github.com/shamwari-ai/shamwari-gateway). The two
   are independent on purpose: do not factor them into a shared package.
2. **Only open-weight output may train Shamwari Mind, and storing is not
   consent.** Core refuses a trainable row without explicit consent.

## Checks

CI (`.github/workflows/ci.yml`) calls the org's reusable workflows:
`reusable-ci-python-piptools.yml` with the import check
`import main, ingest_ground; print(main.app.title, main.app.version)`,
`reusable-gitleaks.yml` and `reusable-codeql.yml` for Python. The PR title is
linted by `pr-title-lint.yml`. There is no Python source on `main` yet, so
there is nothing to run locally today.

## Conventions

- **`requirements.txt` is generated, never hand-edited.** It comes from
  `pip-compile --generate-hashes` over `requirements.in`. CI runs
  `check_lock.py` offline, then installs with `--require-hashes`. The lock is
  resolved for CPython 3.11 on Linux; on macOS or Windows develop against
  `requirements.in`. Dependabot's `pip` ecosystem stays disabled because it
  would strip the hashes.
- **`db/` travels with the service**, never on its own. Do not run or add a
  migration that nothing here tests against.
- **Embeddings are locked to `@cf/baai/bge-m3` at 1024 dimensions.**
  `ingest_ground.py` hard-fails on a mismatch; keep that behaviour.
- **Conventional Commits** on every commit and PR title; the PR title becomes
  the squash-commit subject.
- **Trunk-based.** `main` is the only target; open every pull request against
  it and merge once CI is green.
- **Pin actions to full commit SHAs** when you add or change a workflow that
  uses third-party actions.
- **This repository is public.** No secrets, keys, tokens, connection strings
  or internal hostnames in code, commits, PRs or issues.

## Out of bounds

- Do not touch credentials, database schemas in a live environment, deploys or
  infrastructure from here.
- Do not import history or restructure branches. The extraction is the
  owner's call.
- Do not edit `LICENSE` or `NOTICE`.
