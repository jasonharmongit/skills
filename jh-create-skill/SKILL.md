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
├── reference.md      # optional
├── examples.md       # optional
└── scripts/          # optional
```

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
  - **`dependencies`:** skills this one tells the agent to read and follow (`` **`skill-name` skill** ``, `` **`skill-name`** ``, or a link to `skill-name/SKILL.md`). Skill-to-skill only - not reference files, hooks, or sub-paths.
  - **`dependents`:** the reverse. Keep both sides in sync.
  - Comma-separated `name` values. Omit empty fields.
  - Do not infer dependencies from artifact names or shared tooling unless the body explicitly names the skill.

## Content

Read the **`writing-for-agents`** skill and follow it - this skill covers skill-specific mechanics only.

- No title header or description - frontmatter already supplies identity.
- List a skill in `dependencies` only when this body routes the agent through that skill's workflow; an isolated fact or rule stays inline here.
- When a skill is listed, remove from this body any step or rule that dependency already owns - do not restate it.