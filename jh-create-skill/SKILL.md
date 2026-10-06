---
name: jh-create-skill
description: Guides users through creating effective Agent Skills for Cursor.
disable-model-invocation: true
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
description: Brief high-level description of what this skill does.
disable-model-invocation: true
metadata:
  dependencies: other-skill
  dependents: downstream-skill
---
```

- **name:** kebab-case, max 64 chars, lowercase letters, numbers, hyphens only.
- **description**: A single sentence giving a very high-level summary of what the skill does. Do not include specifics about how the skill works, when to invoke it, or trigger terms.
- **metadata:**
  - **`dependencies`:** skills this one tells the agent to read and follow (`` **`skill-name` skill** ``, `` **`skill-name`** ``, or a link to `skill-name/SKILL.md`). Skill-to-skill only - not hooks or sub-paths.
  - **`dependents`:** the reverse. Keep both sides in sync.
  - Comma-separated `name` values. Omit empty fields.
  - Do not infer dependencies from artifact names or shared tooling unless the body explicitly names the skill.

## Content

Read the **`writing-for-agents`** skill and follow it - this skill covers skill-specific mechanics only.

- Do not include any kind of header. No title, no description, no starting section, no introduction, no preface.
- List a skill in `dependencies` only when this body routes the agent through that skill's workflow; an isolated fact or rule stays inline here.
- When a skill is listed, remove from this body any step or rule that dependency already owns - do not restate it.

### Task, not one run

A skill tells the agent **how to do a kind of task**. It is not a write-up of **one session** you or the agent already ran.

**Put in the skill**

- Ordered workflow and scope limits (what to do, what to skip).
- Hard rules the agent breaks without them (environment traps, "never halt remsh", approval gates).
- Pointers to other skills when that workflow is their job.

**Leave out**

- Procedures that belong to a single feature unless the skill name and description are explicitly about that feature only.
- Copy-paste from a past run: sample message bodies, TypeIDs, SQL, Oban worker names, API routes, Redis keys, "cancel job then reset status" recipes, and similar one-off scripts.
- Details the agent can load at run time from the user's request, tests, or the codebase.

At run time the agent reads the **user's ask** and the **repo** for specifics. The skill stays stable when those specifics change.
