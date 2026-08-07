---
name: add-testing
description: Implements missing test coverage for branch changes.
disable-model-invocation: true
metadata:
  dependencies: code-review
---

## Scope

All changes on the branch - committed, staged, and unstaged:

```bash
git diff --name-status "$(git merge-base HEAD origin/main)"...HEAD
git diff --cached --name-status
git diff --name-status
```

Read full diffs for changed application code (skip lockfiles, generated assets, and pure config unless behavior changed):

```bash
git diff "$(git merge-base HEAD origin/main)"...HEAD
git diff --cached
git diff
```

## Workflow

1. Map changed modules/functions/routes/workers to existing test files. Read those tests and the integration-test happy paths for touched entry points.
2. For each behavior added or changed on the branch, decide whether existing tests already cover it, should be expanded, or need a new test.
3. Skip trivial wiring, type-only changes, or behavior already exercised by nearby tests. Prefer one meaningful assertion over many redundant cases.
4. Implement: add or expand tests in the right files. Match existing style, helpers, and fixtures. Follow [`tests.md`](~/private-skills/surge/skills/code-review/references/tests.md) for how to write tests.
5. Run the affected tests. Fix failures before finishing.

## Summary

After implementation, give a short summary the user can read in 10-15 seconds. Format it however makes sense.

Write in clear, direct, plain language. Say what changed and why. Focus on anything that needs explanation: new tests added, existing tests expanded, gaps intentionally left uncovered, or test run results if relevant. Skip details the user can see in the diff.
