---
name: interview
description: Invoke only when interview is mentioned by name. Locked assumptions and frontier questions asked in rounds before a phased sketch
metadata:
  author: jasonharmon
  dependencies: markdown-plan
---

## Workflow

Lock the open decisions here. The implementation sketch comes later.

The goal is full alignment with the user, ready for implementation — not an interview for its own sake. Rounds and questions run to whatever number that takes: a single round of one question is a complete interview when it settles everything.

1. **Draft locked assumptions** — Extract assumptions from the approach being explored, your research, investigation, and chat history with the user. Start with the primary approach, then add key constraints, scope boundaries, and decisions already made.
   - Very concise: one short phrase or sentence per bullet
   - Also lock **implied decisions**: corollaries that follow from the user's words and the standard shape of this kind of work (see **Implied decisions** below). Record them here; do not ask the user to confirm them
   - Add a bullet beyond what the approach and chat history already establish only when the user asks for it
   - Hold the list for now. Nothing reaches the plan file until step 3 establishes there is a frontier

2. **Gather facts** — Facts are your job, never the user's. Think deeply about what still needs to be decided, then settle every part of it the environment can answer within the **fact boundary**:
   - User-provided context: the current message, attachments, URLs the user sent, and chat history in this conversation
   - The immediate repo: source, tests, docs, and plan files the user attached or named in this conversation
   - Do not read paths outside the immediate repo — including other plan files, other repos, or `~/.cursor/plans` — unless the user explicitly points you there
   - Subagents and codebase search stay inside the fact boundary
   - Placement: load the `rules.md` from `review-layers`, `review-minimalism`, `review-contracts`, and the `review-structuring` skills (but do not follow the `single-review` skill; use them just to reference their rules). Settle layer, defensiveness, helper extraction, peeling, and return shape from those skills. A choice those skills already decide is not a frontier question
   - Put only *decisions* to the user; anything you could look up within the boundary, look up
   - Before treating something as a gap, check whether it is already locked or implied. If yes, do not ask
   - Identify gaps: which decisions would directly affect implementation scope, file count, PR size, or architecture? Which decisions would change how work flows through the system?

3. **Compute the first frontier and write it** — The open decisions form a tree: every decision branches into the decisions that hang off it. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask *now* without guessing at answers you have not heard yet. A question that depends on another question in the same round belongs to a later round.
   - **Frontier bar:** Ask only when a decision genuinely matters to behavior, scope, or architecture and cannot be inferred from context. Assume sensible defaults for low-stakes implementation details and obvious corollaries, and record them under locked assumptions instead of asking
   - Each frontier question should be answerable and material to moving forward; hold edge cases that can wait
   - Before each `Answer:` line, write `Recommendation:` followed by one sentence stating what you would choose and why
   - **Empty frontier → no interview document.** If implied decisions and facts settle every requirement and implementation choice, do not create a `.plan.md`. Skip to step 7
   - Otherwise read the `markdown-plan` skill to write a plan file with an `## Interview` section in the shape of the Example below: `### Locked assumptions` from step 1, then `### Round 1` holding this frontier

4. **Enter discussion mode** — When a frontier exists, write the plan file only. Creating it ends the turn: send no chat message at all, not even its location or a summary. The user may answer questions, ask their own clarifying questions, discuss tradeoffs, or request changes to assumptions.
   - Every question and every answer lives in the `.plan.md` file
   - Recommendations are yours to write; answers arrive only as the user's edits to each `Answer:` line
   - `AskQuestion` and any other tool that surfaces as a chat prompt are off-limits for this skill
   - **Stop here** until **round closure** (below). Do not run step 5, edit the plan to finalize answers, or append a later round.
   - **`next` in chat** is advance approval for steps 5 and 6 only. It is not permission to implement or to move on to a later solutionize step.

**Round closure** — Never close a round without the user's explicit advance approval in chat.

- Advance approval is the user message **`next`** after the round exists. Other wording (`continue`, `go ahead`, `please continue`, system reminders, or a new task) is **not** advance approval.
- Until the user sends **`next`**, the round stays open. Blank `Answer:` lines are unanswered, not acceptance of `Recommendation:`. Do not run step 5, adopt recommendations, or describe recommendations as settled decisions in chat.
- After the user sends **`next`**, blank `Answer:` lines adopt the matching `Recommendation:` in the plan (step 5). That adoption happens only in the plan rewrite triggered by **`next`**, never earlier.
- Closing a round means running step 5 (rewrite that round in the plan) and, if needed, step 6 (append the next round). Do neither before **`next`**.

5. **Review answers and recompute the frontier** — Run this step only after the user's **`next`** since the round under review was written, never on a system reminder or other non-**`next`** message. Re-read the answers and introspect on what they reveal: which questions do they unblock, which gaps do they expose, which assumptions do they settle or change? Gather any new facts they point to within the same fact boundary as step 2.
   - Close out the round under review: for each question, if the user filled `Answer:`, remove its `Recommendation:` line; if they left `Answer:` blank, remove the `Answer:` line and keep `Recommendation:` as the settled decision
   - An ambiguous, contradictory, or incomplete answer becomes a question in the next round rather than a decision you carry forward on your own read of it
   - Each round of answers reshapes the tree: settled decisions push the frontier outward and unblock the questions that depended on them

6. **Ask the next round** — Only after step 5 closes the current round (which requires **`next`** first), write the next `### Round N` block if a frontier remains and repeat steps 4 and 5. When the frontier is empty after closing a round, go to step 7; do not append another round.
   - Append each round at the end using the next index (`### Round 2`, `### Round 3`, and so on); earlier rounds stand as written

7. **Interview over — ask before implementing** — Stop interview work. In chat, ask whether the user would like you to implement. Wait for explicit approval before you edit application code, tests, config, or assets, or before you treat the handoff as proceed to sketch in a solutionize workflow. **`next` is not implementation approval.** If an interview plan exists, do not add anything else to it.

---

## Example

```markdown
## Interview

### Locked assumptions

- Approach focuses on pagination optimization
- Changes apply only to the list view (drill-in view unchanged)
- Uses the existing `status` field for filtering

### Round 1

- Should filtering state persist when navigating between list and drill-in?
  - Recommendation: Persist filter state in the URL query string so back navigation restores the list view the user left.
  - Answer:

- Should filtering be available for export if added later?
  - Recommendation: Design the filter API so export can reuse the same query parameters without duplicating logic.
  - Answer:
```

After step 5 closes a round (`next` with the first question answered, the second left blank):

```markdown
### Round 1

- Should filtering state persist when navigating between list and drill-in?
  - Answer: Yes — persist in the URL query string.

- Should filtering be available for export if added later?
  - Recommendation: Design the filter API so export can reuse the same query parameters without duplicating logic.

### Round 2

- Should retry attempts be included in failed row counts?
  - Recommendation: Exclude retries from the failed count so the metric reflects distinct failures, not recovery attempts.
  - Answer:
```
