---
name: refine-skill
description: Invoke only when refine-skill is mentioned by name. Iterates on a skill by exercising it with sub-agents and refining the instructions.
metadata:
  author: jasonharmon
  dependencies: create-or-edit-skill
---

Refine a skill by running it through a sub-agent loop. Read the `create-or-edit-skill` skill when editing skill content - keep instructions concise and generalizable.

## Inputs

You need:

- **Skill path** - e.g. `~/private-skills/surge/skills/my-skill/SKILL.md`
- **Exercise task** - one concrete job the skill is meant to handle (keep the sub-agent prompt simple)
- **Artifact** (optional) - output the skill produces (file, notebook, etc.) that can be deleted so each run starts fresh

## Confirm with user

Before the first pass, present the inputs and loop plan to the user:

- Skill path and exercise task
- Artifact to reset each pass (if any)
- Up to **3 passes** of reset → exercise → analyze → adjust; stop early on a clean sub-agent report, diminishing returns, or user request

Wait for confirmation before launching pass 1. If inputs are missing or unclear, ask before planning.

## Loop

Repeat until friction is low, the user says stop, or you reach **3 passes** (whichever comes first). One pass = reset, exercise, analyze, adjust.

### 1. Reset

Delete the artifact from the prior iteration if there is one. Do not delete the skill itself.

### 2. Exercise

Spawn a sub-agent using the prompt template below. Use model **`composer-2.5-fast`** (Composer 2.5 Fast) unless the user asks for another model. Keep the exercise task to one sentence; do not over-specify how to do the work.

### 3. Analyze

Read the sub-agent report. If section 3 says there were no skill issues, treat that as success - skip to **Stop** unless you spotted something the sub-agent missed.

Otherwise look for:

| Signal | Skill fix |
| --- | --- |
| Confusion or wrong order | Clarify workflow; one path for new vs update cases |
| Contradictory rules | Remove or reconcile (common in open/sync-style workflows) |
| Repeated inference | Add a general rule, not a one-off example |
| Long detour before real work | Add a "plan first" step or sharper scope |
| Branch/domain specifics in a general skill | Remove; point at how to discover context instead |
| Bloated sections | Cut per create-or-edit-skill |

Do not edit the skill based on sub-agent mistakes that the skill already forbids.

### 4. Adjust

Edit the skill. Prefer:

- Workflow and decision rules over copy-paste templates
- General discovery steps ("read the primary test file for the entry point") over hard-coded names
- Resolving conflicts over adding exceptions

Update `metadata.dependencies` / `dependents` if relationships changed.

### 5. Stop

Stop when:

- The sub-agent completes cleanly with no material skill feedback, or
- The user says stop, or
- You have completed **3 passes**, or
- A pass fixes only nitpicks (diminishing returns)

Summarize what changed and why. Do not spawn another iteration unless the user asks.

## Sub-agent prompt template

```
You are testing whether the `<skill-name>` skill works well. Follow it exactly.

**Context:** <repo, branch, paths - only what the sub-agent needs>

**Your only task:** Read `<skill-path>` (and dependency skills it names). Use it to <exercise task in one sentence>.

In your final response include:
1. Every step you took, in order (note confusion, retries, or dead ends if any)
2. Your final output (<what the skill expects>)
3. Skill feedback - list problems if any, or say **none** if the skill was clear and you have nothing to change

Do not invent skill problems. A report with no issues in section 3 is valid.

<constraints, e.g. Do not commit. Do not run notebook cells.>
```

Replace placeholders; keep the task sentence short.
