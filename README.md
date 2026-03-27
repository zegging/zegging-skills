# zegging-skills

zegging's skills marketplace — a curated collection of Claude skills for code review, architecture analysis, and engineering best practices.

## What is a Skill?

A **skill** is a markdown file (`SKILL.md`) that gives Claude a structured set of instructions, checklists, and guidelines for a specific domain. Skills are activated automatically when relevant topics arise in conversation.

## Skill Template

Use [`skills/template/SKILL.md`](skills/template/SKILL.md) as a starting point for any new skill. Copy the directory, rename it, and fill in the placeholders.

### Directory Structure

```
skills/
└── your-skill-name/
    └── SKILL.md
```

### SKILL.md Frontmatter

Every skill begins with YAML frontmatter:

```yaml
---
name: your-skill-name          # Unique identifier (kebab-case)
description: ...               # When and why to activate this skill
disable-model-invocation: false # Set true to suppress LLM invocation
---
```

### Recommended Sections

| Section | Purpose |
|---|---|
| **Purpose** | One-paragraph goal statement |
| **Core Philosophy** | Guiding principles for decisions within this skill |
| **Checklist** | Categorized checks with success criteria |
| **Common Anti-Patterns** | Known failure modes with fixes |
| **Process** | Step-by-step instructions for the model |
| **Output Format** | Expected structure of the model's response |

## Adding a Skill to the Marketplace

After creating your skill, register it in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) by adding an entry to the `plugins` array:

```json
{
  "name": "your-skill-name",
  "source": "./skills/your-skill-name",
  "description": "...",
  "version": "1.0.0",
  "author": { "name": "your-name" },
  "keywords": ["keyword1", "keyword2"]
}
```

## License

MIT
