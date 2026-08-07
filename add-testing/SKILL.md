---
name: add-testing
description: Implements missing test coverage for branch changes.
disable-model-invocation: true
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
4. Implement: add or expand tests in the right files. Match existing style, helpers, and fixtures.
5. Run the affected tests. Fix failures before finishing.

## Summary

After implementation, give a high-level summary:

- **Coverage added** - one bullet per test file touched: what behaviors were exercised (not every assertion)
- **Skipped** - brief note if anything on the branch had no test gap worth filling
- **Test run** - command run and pass/fail outcome

Keep the summary concise; no code dumps or full test listings.
