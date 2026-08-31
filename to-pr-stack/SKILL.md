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
- Stacking is **organizational only**. Earlier slices may **exclude** hunks or files that depend on work in later slices (for example, omit an enqueue until the worker lands). What is prohibited is **modifying, adding, or stubbing** code in intermediate slices: no rewrites, no stand-ins, no new logic. Every line in a slice must be an exact, unchanged excerpt from the existing work; excluded lines appear unchanged in a later slice.
- Always use a linear `gh stack`, even when two slices have no compile-time dependency and could ship as parallel PRs off `main`. Order foundations before consumers when dependencies exist; otherwise order by reviewer concern or logical grouping.

## 1. Gather context

Compare all work (committed on the branch plus unstaged and staged) against the default branch.

Recover intent from branch name, Linear/git issue links, chat history, and diff themes.

Read `gh stack --help` if you are unsure of a command flag.

## 2. Propose the split

Respond with **nested markdown only**. No other prose, diagrams, or commentary.

Show stack order bottom to top. Position `1` is always the bottom slice, targeting `main`. Every slice gets a new branch name; do not reuse the current branch.

For each slice, use the heading as the PR title: short and descriptive, starting with a verb (for example `show`, `add`, `persist`, `implement`, `fix`). Do not use branch names as titles. Do not repeat titles in a separate list after the outline.

Branch names use the slice position, a hyphen, then a kebab-case slug from the title (for example position `1` with title `Add foo bar` becomes `1-add-foo-bar`). Derive them during execution; do not include branch names in the proposal or final report.

```markdown
### 1 - Add foo bar

File count: x

- path/to/file.ex
- path/to/other.ex:start-end
```

List every path in the slice as one bullet. For intra-file splits, use `path:start-end` line ranges instead of the full path. File count is the number of bullets.

Optimize for **ease of review**, not ease of splitting. Full-file slices are common; split within a file when that helps review and each slice uses exact excerpts from the existing work. Exclude dependent hunks from earlier slices rather than rewriting them. Keep tightly coupled changes together when they cannot be separated by exclusion alone.

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

**Initialize** on an updated default branch. Create every slice as a new branch from the approved plan:

```bash
git checkout main && git pull origin main
gh stack init --base main 1-add-foo-bar 2-update-baz-qux ...
```

Branch names bottom to top, derived from each slice's position and title. The bottom branch targets `main`; each upper branch targets the one below. Leave the original branch unchanged.

**Per slice** (bottom up):

1. `gh stack bottom` or `gh stack up` to reach the slice branch.
2. If the branch tip is behind the parent slice, fast-forward: `git merge <parent-branch> --ff-only`.
3. Restore this slice's changes from `stash@{0}`. Use whole-file checkout when the slice is file-complete; use `git add -p` or patch staging when the approved plan splits within a file. Do not edit restored content.
4. For untracked files in the stash, use the stash untracked parent: `git checkout stash@{0}^3 -- <paths>`.
5. Stage deletions explicitly (`git rm`) when a slice renames or removes files.
6. Commit with a message focused on why.
7. Repeat on the next branch up.

**Publish**:

```bash
gh stack submit --auto --open
```

Leave every PR description blank. After submit, clear any auto-filled body:

```bash
gh pr edit <pr-number> --body ""
```

## 4. Report back

After execution, give a short summary: PR numbers, titles, URLs, base branch for each, and whether the working tree is clean. Mention stash and backup ref if they still exist. Do not delete backup refs unless the user asks.
