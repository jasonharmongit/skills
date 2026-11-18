---
name: fix-skill
description: Invoke only when fix-skill is mentioned by name. Applies user feedback to a skill invoked earlier in the conversation, then offers to apply the same feedback to the current task.
metadata:
  author: jasonharmon
  dependencies: writing-for-agents
---

## Workflow

1. **Select target skill.** Find the skill invoked earlier in this conversation (`/skill-name`, manual attachment, or explicit `SKILL.md` read). If more than one skill was used, pick the one the feedback is about. If unclear, ask before editing.

2. **Find target skill path.** Search for `SKILL.md` with a matching `name` in frontmatter:
   - `~/private-skills`
   - `~/skills`

   **Off-limits:** `~/.agents`. Do not read, search, or edit anything there.

   If no match, tell the user which repos you searched and ask for the skill path or name.

3. **Apply feedback to target skill.** Read the `writing-for-agents` skill before editing. Edit only the resolved `SKILL.md` and sibling reference files in the same skill folder. Apply the user's feedback directly. Keep unrelated content unchanged. Update `metadata.dependencies` / `dependents` if relationships changed. When feedback becomes a rule or guideline, **generalize** it: one short principle, not a long or chat-specific formulation. Omit domain examples, synonym lists, and one-off case detail unless the user asks to keep them.

4. **Validate the edit.** Re-read the `writing-for-agents` skill. Check your changed lines against it (DRY, phrasing, pruning, length). If a new rule still reads long or tied to one incident, generalize or shorten it again. Do not reply to the user until this pass is done.

5. **Offer to apply feedback to current situation.** After the skill edit, offer to do any concrete actions that apply the same feedback to the current situation: a code edit, a re-output in chat, a re-run of an earlier step, or another action that fits. Wait for explicit approval before acting.

6. **Apply feedback to current situation.** On approval, do the offered action.
