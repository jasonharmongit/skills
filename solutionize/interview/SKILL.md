---
name: interview
description: Locked assumptions and frontier questions asked in rounds before a phased sketch
disable-model-invocation: true
metadata:
  dependencies: markdown-plan
  dependents: solutionize
---

## Workflow

Lock the open decisions here. The implementation sketch comes later.

The goal is full alignment with the user, ready for implementation — not an interview for its own sake. Rounds and questions run to whatever number that takes: a single round of one question is a complete interview when it settles everything.

1. **Draft locked assumptions** — Extract assumptions from the approach being explored, your research, investigation, and chat history with the user. Start with the primary approach, then add key constraints, scope boundaries, and decisions already made.
   - Very concise: one short phrase or sentence per bullet
   - Add a bullet beyond what the approach and chat history already establish only when the user asks for it
   - Hold the list for now. Nothing reaches the plan file until step 3 establishes there is a frontier

2. **Gather facts** — Facts are your job, never the user's. Think deeply about what still needs to be decided, then settle every part of it the environment can answer within the **fact boundary**:
   - User-provided context: the current message, attachments, URLs the user sent, and chat history in this conversation
   - The immediate repo: source, tests, docs, and plan files the user attached or named in this conversation
   - Do not read paths outside the immediate repo — including other plan files, other repos, or `~/.cursor/plans` — unless the user explicitly points you there
   - Subagents and codebase search stay inside the fact boundary
   - Put only *decisions* to the user; anything you could look up within the boundary, look up
   - Identify gaps: which decisions would directly affect implementation scope, file count, PR size, or architecture? Which decisions would change how work flows through the system?

3. **Compute the first frontier and write it** — The open decisions form a tree: every decision branches into the decisions that hang off it. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask *now* without guessing at answers you have not heard yet. A question that depends on another question in the same round belongs to a later round.
   - Each frontier question should be answerable and material to moving forward; hold edge cases and micro-decisions that can wait
   - Before each `Answer:` line, write `Recommendation:` followed by one sentence stating what you would choose and why
   - **An empty frontier means no interview.** If the approach leaves no open decision on requirements or implementation, state that in chat, write nothing, and stop for the user's direction
   - Otherwise read [markdown-plan/SKILL.md](../../markdown-plan/SKILL.md) to write a plan file with an `## Interview` section in the shape of the Example below: `### Locked assumptions` from step 1, then `### Round 1` holding this frontier

4. **Enter discussion mode** — Write the plan file only; send no chat message. The user may answer questions, ask their own clarifying questions, discuss tradeoffs, or request changes to assumptions.
   - Every question and every answer lives in the `.plan.md` file
   - Recommendations are yours to write; answers arrive only as the user's edits to each `Answer:` line
   - `AskQuestion` and any other tool that surfaces as a chat prompt are off-limits for this skill
   - **Stop here.** Do not close the round, open the next round, or assume blank `Answer:` lines mean acceptance. Wait for the user to edit the plan and give explicit permission to advance the interview.

5. **Review answers and recompute the frontier** — Re-read the answers and introspect on what they reveal: which questions do they unblock, which gaps do they expose, which assumptions do they settle or change? Gather any new facts they point to within the same fact boundary as step 2.
   - Close out the round under review: for each question, if the user filled `Answer:`, remove its `Recommendation:` line; if they left `Answer:` blank, remove that line and keep `Recommendation:` as the settled decision
   - An ambiguous, contradictory, or incomplete answer becomes a question in the next round rather than a decision you carry forward on your own read of it
   - Each round of answers reshapes the tree: settled decisions push the frontier outward and unblock the questions that depended on them

6. **Ask the next round** — Only after step 5 closes out the current round with explicit user permission, write the next `### Round N` block if a frontier remains and repeat steps 4 and 5. Once it is empty, go to step 7.
   - Append each round at the end using the next index (`### Round 2`, `### Round 3`, and so on); earlier rounds stand as written

7. **Ready for approval** — Do not add anything else to the plan file. Tell the user you are ready to proceed, and wait for their explicit confirmation before implementing.

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

After step 5 closes a round (user answered the first question, left the second blank):

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
