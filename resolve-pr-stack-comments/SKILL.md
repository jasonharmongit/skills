---
name: resolve-pr-stack-comments
description: Triage duplicate review comments across a monolithic PR and its split stack.
disable-model-invocation: true
---

## Constraints

Do not change code, commit, push, or rebase.

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

Open that PR with `gh pr view`. Build a **monolith ledger** (internally):

- Full diff: `gh pr diff` or `git diff "$(git merge-base HEAD origin/main)"...HEAD`
- Every discussion comment and every inline review thread (resolved and unresolved)
- Which concerns were already fixed, explicitly ignored, or deferred in chat on the monolith PR

Skim title, body, and changed paths. This ledger is the source of truth for duplicate detection.

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

### Evaluate duplicates

For each unresolved comment, mark **duplicate** when any of these hold:

1. **Monolith ledger** - the same concern was already fixed or explicitly ignored on the monolith PR (including resolved inline threads and discussion replies).
2. **Narrow scope** - the comment targets code or behavior absent from this slice's diff but present in the monolith diff (or a higher stack layer that the monolith includes). Use `gh pr diff <stack-pr-number>` for the slice and the monolith diff for the full change set. Typical cases: "missing handler", "missing migration", "missing test", "unused import until later slice".

When unsure, leave the comment open.

### Close duplicates

Track counts per branch: `closed` and `remain open`.

**Inline review thread**

1. Reply:

```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: $threadId, body: "duplicate"}) {
    comment { id }
  }
}' -f threadId=PRRT_...
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
<summary>duplicate</summary>

${ORIGINAL_BODY}
</details>
EOF
)"
```

Do not edit comments that are not duplicates.

## 4. Report

Brief summary table:

| Branch | PR | Closed (duplicate) | Remain open |
|--------|----|--------------------|-------------|
| `1-add-foo-bar` | [#1802](url) | n | n |
| … | … | … | … |

List any stack PRs that had zero unresolved comments.
