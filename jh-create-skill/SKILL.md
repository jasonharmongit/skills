---
name: jh-create-skill
description: >-
  Guides users through creating effective Agent Skills for Cursor. Use when you
  want to create, write, or author a new skill, or asks about skill structure,
  best practices, or SKILL.md format.
---

## Skill Anatomy

### Frontmatter

Every `SKILL.md` file starts with YAML frontmatter:

```markdown
---
name: your-skill-name
description: Brief high-level description of what this skill does.
disable-model-invocation: true
---
```

- **name**: Kebab-case identifier for the skill
- **description**: A single sentence giving a very high-level summary of what the skill does. Do not include specifics about how the skill works, when to invoke it, or trigger terms.
- **disable-model-invocation**: Include `disable-model-invocation: true` by default so the skill only loads when explicitly named.

**Skill connections** (optional, in `metadata`):

```yaml
metadata:
  dependencies: other-skill, another-skill
  dependents: downstream-skill
```

- **`dependencies`**: Other skills this one tells the agent to read or follow (explicit `` **`skill-name` skill** ``, `` **`skill-name`** ``, or a link to `skill-name/SKILL.md`). Skill-to-skill only - not reference files, hooks, or sub-paths under another skill.
- **`dependents`**: The reverse - skills that depend on this one. Keep both sides in sync when you add or change a link.
- Use comma-separated skill `name` values. Omit `dependencies` or `dependents` when empty.
- Do not infer dependencies from artifact names or shared tooling unless the body explicitly names the skill.

When editing a skill, check its `metadata` and update connected skills if the relationship changed.

### Body

Do not include a header on the skill. The frontmatter already supplies a title and high-level description, so that isn't needed. For the body, jump straight into the skill itself.

## Skill File Structure

### Directory Layout

Skills are stored as directories containing a `SKILL.md` file:

```
skill-name/
├── SKILL.md              # Required - main instructions
├── reference.md          # Optional - detailed documentation
├── examples.md           # Optional - usage examples
└── scripts/              # Optional - utility scripts
    ├── validate.py
    └── helper.sh
```

## Core Authoring Principles

### 1. Concise is Key

The context window is shared with conversation history, other skills, and requests. Every token competes for space.

**Default assumption**: The agent is already very smart. Only add context it doesn't already have.

Challenge each piece of information:
- "Does the agent really need this explanation?"
- "Can I assume the agent knows this?"
- "Does this paragraph justify its token cost?"