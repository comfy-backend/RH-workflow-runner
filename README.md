# RH-workflow CI runner

This repository is a **compute-only runner**: it hosts GitHub Actions workflows
that execute the scheduled jobs of a private data pipeline. It intentionally
contains *nothing but workflow definitions* — no source code, no data, and no
credentials. (Public repos get unlimited Actions minutes, which is why the
compute moved here.)

## Workflows

| Workflow | Trigger | Purpose |
|---|---|---|
| `refresh-catalog.yml` | schedule (daily + weekly) / dispatch / push | The production pipeline: checks out a **private** source repo, runs the refresh, pushes results to **private** data/code repos, files alert issues in the private repo. On push it runs the test gate only. |
| `smoke.yml` | push / dispatch | Connectivity self-test: private-repo checkouts, secret presence, GitLab auth, cross-repo issue-write path. |
| `ci.yml` | `repository_dispatch` (`code-push`) / dispatch | Runs the private repo's test suite for a pushed SHA (triggered from the private repo after each push). |

## Security posture

- All credentials (the GitHub PAT used for private checkouts/pushes, the GitLab
  token, Turso tokens) live as **encrypted GitHub Actions secrets** of this
  repo — never in any file here.
- This repo is public, so **run logs are public**: every secret value is
  masked automatically by GitHub's log redaction (registered secret values are
  redacted wherever they appear, including inside URLs).
- No `pull_request` triggers exist on the pipeline workflows, so fork PRs can
  never access secrets.
- Every private-repo access authenticates with the PAT secret; the default
  `GITHUB_TOKEN` of this repo cannot read or write anything private.
