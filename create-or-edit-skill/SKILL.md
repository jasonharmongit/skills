---
name: create-or-edit-skill
description: How to create and edit skills.
metadata:
  dependencies: writing-for-agents
  dependents: refine-skill
---

## Structure

### File structure

```
skill-name/
├── SKILL.md
└── scripts/          # optional
```

Put every step, rule, and example in **`SKILL.md`**. Do not add other sibling docs.

### Frontmatter

```yaml
---
name: your-skill-name
description: Invoke only when your-skill-name is mentioned by name. Summarizes what the skill does in third-person present tense.
metadata:
  dependencies: other-skill
  dependents: downstream-skill
---
```

- **name:** kebab-case, max 64 chars, lowercase letters, numbers, hyphens only.
- **description:** Do **not** use `disable-model-invocation`. It blocks discovery when another skill references this skill by name. For skills invoked only via `/skill-name`, explicit attachment, or a named pointer in another skill body, start with `Invoke only when {name} is mentioned by name. ` (use the skill's `name` value). Follow with a high-level summary in **third-person present tense** (e.g. `Investigates`, `Reviews`, `Runs` - not `Investigate`, `Review`, `Run`). You may add a second sentence starting with `Uses when` for extra natural-language triggers. Do not restate the full workflow. For skills the agent should auto-select from ordinary user phrasing (uncommon), omit the invoke prefix and write the summary in third person; you may include `Use when` trigger hints.
- **metadata:**
  - **`dependencies`:** skills this one tells the agent to read and follow. Skill-to-skill only - not hooks or sub-paths. List each skill's `name` here; in the body, point to it in natural language (see **Content**).
  - **`dependents`:** the reverse. Keep both sides in sync.
  - Comma-separated `name` values. Omit empty fields.
  - Do not infer dependencies from artifact names or shared tooling unless the body explicitly names the skill.

## Content

Read the `writing-for-agents` and follow it - this skill covers skill-specific mechanics only.

- Do not include any kind of header. No title, no description, no starting section, no introduction, no preface.
- List a skill in `dependencies` only when this body routes the agent through that skill's workflow; an isolated fact or rule stays inline here.
- When a skill is listed, remove from this body any step or rule that dependency already owns - do not restate it.
- When the body routes the agent through a dependency skill, write it in natural language: read and follow the `<skill-name>` skill. Use that skill's `name` in backticks, always with the word skill. Do not use file paths.

### Task, not one run

A skill tells the agent **how to do a kind of task**. It is not a write-up of **one session** you or the agent already ran.

**Put in the skill**

- Ordered workflow and scope limits (what to do, what to skip).
- Hard rules the agent breaks without them (environment traps, "never halt remsh", approval gates).
- Pointers to other skills when that workflow is their job (read and follow the `<skill-name>` skill).

**Leave out**

- Procedures that belong to a single feature unless the skill name and description are explicitly about that feature only.
- Copy-paste from a past run: sample message bodies, TypeIDs, SQL, Oban worker names, API routes, Redis keys, "cancel job then reset status" recipes, and similar one-off scripts.
- Details the agent can load at run time from the user's request, tests, or the codebase.

At run time the agent reads the **user's ask** and the **repo** for specifics. The skill stays stable when those specifics change.
