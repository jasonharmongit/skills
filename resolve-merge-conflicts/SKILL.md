---
name: resolve-merge-conflicts
description: Syncs the current branch with its merge base (stack parent or main), resolves or surfaces conflicts, then runs doctor and pushes.
metadata:
  dependencies: doctor
---

## Step 1 — Detect the base branch

A **stack** PR targets another feature branch, not `main`. Merging `main` when the PR is stacked leaves GitHub conflicts unchanged.

From the repo root, identify the base:

~~~bash
gh stack view
~~~

If the current branch has an open PR, confirm the base:

~~~bash
gh pr view --json baseRefName,mergeable,mergeStateStatus
~~~

- `baseRefName` is not `main` → **stacked**. The base is that branch (the parent PR below this one in `gh stack view`).
- No PR, or `baseRefName` is `main` → base is `main`.

Record the base branch name as `<base>`.

Check the whole **stack** is **linear**. GitHub requires every branch to sit directly on the tip of the branch below it. `gh stack view` marks a broken link with `⚠`. A PR can be `MERGEABLE` while the stack still shows "branches have diverged".

If any link is broken, fix **downstack first** (parent before child). A lower branch that skipped its parent (e.g. petal-components missing a commit from admin-search-lookups) must be rebased and pushed before rebasing branches above it. Rebasing only the top branch leaves duplicate commits and the divergence banner.

When syncing the whole stack, repeat Steps 2–7 **one branch at a time** — each branch onto its updated parent, then push — before moving up.

## Scope — current branch only

Work on the **checked-out branch** only. Sync with `<base>`, resolve its conflict markers, finish the rebase or merge, push.

## Step 2 — Sync with the base

Fetch the base, then sync. For a **stacked** branch, **rebase** onto the parent. For a branch based on `main`, **merge** `main`.

**Stacked** (`<base>` is not `main`):

~~~bash
git fetch origin <base>
git rebase origin/<base>
~~~

**Not stacked** (`<base>` is `main`):

~~~bash
git fetch origin main
git merge origin/main
~~~

If sync completes cleanly, skip to Step 5. If a rebase or merge is still in progress with conflicts, continue to Step 3.

## Step 3 — Find conflicts

~~~bash
git diff --name-only --diff-filter=U
~~~

Read each file's conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) and understand what each side changed. During a rebase, **ours** is the base branch (`<base>`); **theirs** is the commit being replayed.

## Step 4 — Resolve or surface

For each conflict, decide:

**Auto-resolve** when both sides' behavior can stay intact — e.g. both sides add different imports, aliases, or clauses; non-overlapping additions in the same file; one side reformats while the other changes logic elsewhere. Combine both sides, remove markers, and stage the file.

**Surface to the user** when it is a true conflict — the same behavior is implemented differently, one side removes or renames what the other still depends on, or merging both would change runtime semantics. Do not guess. Stop and report:

- File path and approximate location
- What `<base>` does vs what this branch does
- Why both cannot coexist without choosing one behavior

If any true conflicts remain, do not finish the sync. Ask the user how to proceed. Skip Steps 5–7 until conflicts are resolved.

When all conflicts are auto-resolved:

~~~bash
git add <resolved-files>
~~~

Then finish the in-progress operation:

- **Rebase:** `git rebase --continue`
- **Merge:** `git commit --no-edit`

## Step 5 — Doctor

Find, read and follow the `doctor` skill. Fix failures only when directly caused by conflict resolution or by changes in **this branch's diff**. If a failure needs edits owned by a downstream PR, it is out of scope — report it and continue to push unless the user says otherwise. If a failure reflects an unresolved semantic choice from the sync, stop and report to the user instead of guessing.

## Step 6 — Commit and push

Check for uncommitted changes (doctor fixes, conflict resolutions not yet committed, etc.):

~~~bash
git status --short
~~~

If anything is unstaged or staged, commit it with a message that reflects the work (e.g. rebase resolution, doctor fixes).

Push the branch. Rebase rewrites history, so a stacked branch needs a lease push:

~~~bash
# after rebase (stacked branch)
git push --force-with-lease

# after merge (main-based branch)
git push -u origin HEAD   # use -u when there is no upstream yet; otherwise plain git push
~~~

Do not push until doctor has passed and all intended changes are committed.

## Step 7 — Verify on GitHub

Confirm the PR and the stack are healthy:

~~~bash
gh pr view --json mergeable,mergeStateStatus
gh stack view
~~~

`mergeable` should be `MERGEABLE`. No `⚠` on any stack branch. `mergeStateStatus` of `BLOCKED` can mean CI or review requirements, not merge conflicts.

## Notes

- Never reset or abort a rebase/merge unless the user asks.
- Use CLI rebase only (`git rebase origin/<base>`). Never open GitLens interactive rebase or run `git rebase -i`.
- Prefer preserving both sides' intent over picking one side when combination is safe.
- If sync fails for a reason other than conflicts (e.g. uncommitted changes blocking it), fix or report that first.
- `gh stack rebase` rebases the whole stack and can fail on the top branch. When it does, rebase each broken branch manually in downstack order, push each with `--force-with-lease`, then rebase the next branch up.
