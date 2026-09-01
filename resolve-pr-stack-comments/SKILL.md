---
name: resolve-pr-stack-comments
description: Resolve review comments across a pr stack
disable-model-invocation: true
---

## Constraints

Steps 1-4 only: do not change code, commit, push, or rebase.

## Stack conventions

Split stacks are linear: position **1** is the bottom slice (PR base is `main`); each layer above targets the branch below. Branch names follow `<position>-<kebab-case-slug>` (for example `1-add-foo-bar`, `2-parse-us-port-emails`).

After a stack split, the monolith PR should have a comment `review-stack:<number>` pointing at the GitHub stack object.

## 1. Monolith

Confirm the current branch is the **monolithic branch** (full change set, single PR against `main`):

```bash
git branch --show-current
gh pr list --head "$(git branch --show-current)" --json number,title,url,baseRefName,headRefName,state
```

The result must be an open PR whose `baseRefName` is `main`. If not, stop and ask the user to switch to the monolithic branch.

Record the monolithic branch name (`MONOLITH_BRANCH`) for Step 5.

Open that PR with `gh pr view`. Build a **monolith ledger** (internally):

- Full diff: `gh pr diff` or `git diff "$(git merge-base HEAD origin/main)"...HEAD`
- Every discussion comment and every inline review thread (resolved and unresolved)
- Which concerns were already fixed, explicitly ignored, or deferred in chat on the monolith PR

Skim title, body, and changed paths. This ledger is the source of truth for not-applicable detection.

## 2. Stack discovery

The monolith PR is not in the stack. Load the stack the monolith points at:

```bash
REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
```

Search the monolith PR body and discussion comments for `review-stack:<number>`. When found:

```bash
gh api "repos/${REPO}/stacks/<number>"
```

The response `pull_requests` array is ordered **bottom to top**. Each entry has PR `number`, `head.ref` (branch), and `state`. Record each layer's PR number, branch, and URL (`https://github.com/${REPO}/pull/<number>`).

When no `review-stack:` marker exists, stop and ask the user for the stack number.

## 3. Triage loop

Process stack PRs **one at a time, bottom to top**.

### Load comments

- **Discussion comments**: `gh pr view <number> --comments` or `gh api repos/{owner}/{repo}/issues/{number}/comments`
- **Inline review threads**: GraphQL (keep each thread's `PRRT_…` id):

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $number) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          path
          line
          comments(first: 20) {
            nodes { body author { login } }
          }
        }
      }
    }
  }
}' -f owner=OWNER -f repo=REPO -F number=NUMBER
```

Only triage threads where `isResolved` is false.

### Evaluate not-applicable comments

For each unresolved comment, mark **not applicable** when any of these hold. Use the matching reply text exactly:

1. **Monolith ledger** - the same concern was already fixed or explicitly ignored on the monolith PR (including resolved inline threads and discussion replies). Reply: `not applicable - already addressed`
2. **Narrow scope** - the comment targets code or behavior absent from this slice's diff but present in the monolith diff (or a higher stack layer that the monolith includes). Use `gh pr diff <stack-pr-number>` for the slice and the monolith diff for the full change set. Typical cases: "missing handler", "missing migration", "missing test", "unused import until later slice". Reply: `not applicable - broader context solves this`

When unsure, leave the comment open.

### Close not-applicable comments

Track counts per branch: `closed` and `remain open`.

**Inline review thread**

1. Reply with the matching text from **Evaluate not-applicable comments**:

```bash
gh api graphql -f query='
mutation($threadId: ID!, $body: String!) {
  addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: $threadId, body: $body}) {
    comment { id }
  }
}' -f threadId=PRRT_... -f body="<NOT_APPLICABLE_REPLY>"
```

2. Resolve:

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: {threadId: $threadId}) {
    thread { isResolved }
  }
}' -f threadId=PRRT_...
```

**Discussion (issue) comment**

Edit the original comment, wrapping its existing body:

```bash
gh api --method PATCH "repos/${REPO}/issues/comments/${COMMENT_ID}" \
  -f body="$(cat <<EOF
<details>
<summary><NOT_APPLICABLE_REPLY></summary>

${ORIGINAL_BODY}
</details>
EOF
)"
```

Edit only comments marked not applicable.

## 4. Report

Output **only** the summary table. No heading, prose, bullet lists, or other report content. After the table, say exactly: `Say 'next' to proceed to resolving remaining comments, branch by branch.`

| Branch | PR | Closed (not applicable) | Remain open |
|--------|----|--------------------|-------------|
| `1-add-foo-bar` | [#1802](url) | n | n |
| … | … | … | … |

**Stop here.** Wait for the user to reply `next`.

## 5. Resolve remaining comments

When the user replies `next`, work through stack PRs **bottom to top**. Skip PRs with zero remaining open comments.

Stay checked out on **`MONOLITH_BRANCH`** for the whole step. The user sees every change in full monolith context. Do not check out a stack branch for edits or review laps.

For each stack PR that still has open comments:

1. Read and follow the **`resolve-pr-comments`** skill, using **that stack PR** as the comment target (not the monolith PR).
2. At **`resolve-pr-comments` Step 6**, commit and push on **both** branches before moving on:
   - Commit on `MONOLITH_BRANCH` and push it.
   - Port the same commit(s) onto that stack PR's branch, push it, then run upstack propagation (`gh stack rebase --upstack`, `gh stack push`).
   - Return to `MONOLITH_BRANCH` before the next stack PR.
3. Do not start the next stack PR until the current one is fully finished on GitHub.

When every stack PR has no remaining open comments, tell the user the stack is done.
