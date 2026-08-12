---
name: to-pr-stack
description: Split current work into a GitHub stacked PR series using gh stack.
disable-model-invocation: true
metadata:
  dependencies: surge-rules
---

## Hard rules

- Do not create branches, commit, push, or open PRs until the user approves the split plan.
- Never discard user work. No destructive git commands (`reset --hard`, `clean -fdx`, branch deletion, force-push, history rewrite) without explicit approval.
- Always save a recoverable snapshot before moving work around.
- Stage only named files or hunks. No `git add .` / `git add -A`.
- Always use a linear `gh stack`, even when two slices have no compile-time dependency and could ship as parallel PRs off `main`. Order foundations before consumers when dependencies exist; otherwise order by reviewer concern or logical grouping.

## 1. Gather context

Compare all work (committed on the branch plus unstaged and staged) against the default branch.

Recover intent from branch name, Linear/git issue links, chat history, and diff themes.

Read `gh stack --help` if you are unsure of a command flag.

## 2. Propose the split

Respond with **one concise table only**. No other prose, headers, diagrams, or commentary.

Table columns: stack position (`#`), PR title, file count, one-line scope. Show stack order bottom to top. Position `#1` is always the current branch name (the base PR targeting `main`).

Optimize for **ease of review**, not ease of splitting. Full-file slices are common, but split within a file when that makes each PR easier to review. Keep tightly coupled changes together; minimize unrelated diff per slice.

Wait for explicit approval before executing. A request by the user to make an adjustment is NOT approval - respond with the adjustment, and re-seek approval.

## 3. Execute the stack

**Snapshot** (when work is uncommitted):

```bash
SHA=$(git stash create "pre-split")
if [ -n "$SHA" ]; then
  git update-ref "refs/backup/pre-split-$(date +%s)" "$SHA"
fi
git stash push -u -m "pre-split"
```

**Initialize** on an updated default branch. Use the current branch as the bottom (first) PR:

```bash
BASE_BRANCH=$(git branch --show-current)
git checkout main && git pull origin main
gh stack init --base main "$BASE_BRANCH" <branch-2> ... <branch-n>
```

Branch names bottom to top. The current branch is always position 1; it targets `main`. Each upper branch targets the one below.

**Per slice** (bottom up):

1. `gh stack bottom` or `gh stack up` to reach the slice branch.
2. If the branch tip is behind the parent slice, fast-forward: `git merge <parent-branch> --ff-only`.
3. Restore this slice's changes from `stash@{0}`. Use whole-file checkout when the slice is file-complete; use `git add -p` or patch staging when the approved plan splits within a file.
4. For untracked files in the stash, use the stash untracked parent: `git checkout stash@{0}^3 -- <paths>`.
5. Stage deletions explicitly (`git rm`) when a slice renames or removes files.
6. Commit with a message focused on why.
7. Repeat on the next branch up.

**Publish**:

```bash
gh stack submit --auto --open
```

## 4. Report back

After execution, give a short summary: PR numbers, titles, URLs, base branch for each, and whether the working tree is clean. Mention stash and backup ref if they still exist. Do not delete backup refs unless the user asks.
