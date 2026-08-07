---
name: propose-tests
description: Proposes missing test coverage for branch changes without writing tests.
disable-model-invocation: true
---

Propose tests only. Do not write, edit, or run tests unless the user asks.

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
3. Skip proposals for trivial wiring, type-only changes, or behavior already exercised by nearby tests. Prefer one meaningful assertion over many redundant cases.
4. Output suggestions in the format below. Omit test files with nothing to add or change.

## Output format

Three levels: test file → `describe` → proposed test.

| Level | Heading | Content |
| --- | --- | --- |
| Test file | `##` | Basename only - `billing_test.exs`, not the full path |
| Describe | `###` | Existing or new `describe` string the test belongs in |
| Test | `-` bullet | `test "name"`: one-line note |

**Bullet rules:**

- **New test** - state what it would assert or exercise.
- **Existing test** - say how it would change, not repeat what it already does.
- **Similar case** - reference a sibling: `same as "upgrades plan" but downgrade mid-cycle`.
- Stay concise; plain language; no redundant setup detail.

Omit test files and `describe` blocks with nothing to add or change.

**Example:**

```markdown
## billing_test.exs
### calculate_proration/2
- prorates mid-cycle upgrade: new - credit equals unused days on old plan
- same as "prorates mid-cycle upgrade" but downgrade: new

### apply_coupon/2
- rejects expired coupon: expand - add case for newly added grace-period window

## user_controller_test.exs
### POST /api/users
- returns 422 when email is duplicate: existing - also assert username uniqueness error
```
