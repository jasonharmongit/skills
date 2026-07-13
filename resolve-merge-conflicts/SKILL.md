---
name: resolve-merge-conflicts
description: Merges main into the current branch and resolves or surfaces merge conflicts.
---

# Resolve Merge Conflicts

Merge `origin/main` into the current branch, resolve conflicts that preserve both sides' behavior, surface true conflicts to the user, run doctor, then commit and push.

## Step 1 — Merge main

From the repo root:

~~~bash
git fetch origin main
git merge origin/main
~~~

If the merge completes cleanly, skip to Step 4. If the merge itself created a commit, you still run Steps 4–5.

## Step 2 — Find conflicts

List conflicted files:

~~~bash
git diff --name-only --diff-filter=U
~~~

Read each file's conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) and understand what each side changed.

## Step 3 — Resolve or surface

For each conflict, decide:

**Auto-resolve** when both branches' behavior can stay intact — e.g. both sides add different imports, aliases, or clauses; non-overlapping additions in the same file; one side reformats while the other changes logic elsewhere. Combine both sides, remove markers, and stage the file.

**Surface to the user** when it is a true conflict — the same behavior is implemented differently, one side removes or renames what the other still depends on, or merging both would change runtime semantics. Do not guess. Stop merging that file and report:

- File path and approximate location
- What `main` does vs what this branch does
- Why both cannot coexist without choosing one behavior

If any true conflicts remain, **do not commit the merge**. Ask the user how to proceed. Skip Steps 4–5 until conflicts are resolved.

When all conflicts are auto-resolved:

~~~bash
git add <resolved-files>
git commit --no-edit   # completes the merge if one was in progress
~~~

## Step 4 — Doctor

Read and follow `doctor/SKILL.md`, but **skip Step 1 (Conventions review)**. Run doctor steps 2–4 on the branch as it exists after the merge (including any conflict resolutions).

If doctor fails on something introduced by conflict resolution, fix it within doctor's scope rules. If a failure reflects an unresolved semantic choice from the merge, stop and report to the user instead of guessing.

## Step 5 — Commit and push

Check for uncommitted changes (doctor fixes, conflict resolutions not yet committed, etc.):

~~~bash
git status --short
~~~

If anything is unstaged or staged, commit it with a message that reflects the work (e.g. merge resolution, doctor fixes). Then push the branch to its remote tracking branch:

~~~bash
git push -u origin HEAD
~~~

Use `-u` when the branch has no upstream yet; otherwise a plain `git push` is fine.

Do not push until doctor has passed and all intended changes are committed.

## Notes

- Never force-push, reset, or abort the merge unless the user asks.
- Prefer preserving both sides' intent over picking one side when combination is safe.
- If `git merge` fails for a reason other than conflicts (e.g. uncommitted changes blocking merge), fix or report that first.
