---
name: jh-create-skill
description: >-
  Guides users through creating effective Agent Skills for Cursor. Use when you
  want to create, write, or author a new skill, or asks about skill structure,
  best practices, or SKILL.md format.
---

# Create Skill

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