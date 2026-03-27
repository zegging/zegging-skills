# Skill Authoring SOP

This document captures the skill authoring workflow and formatting guidance that was previously kept in the repository README.

## What is a Skill?

A **skill** is a markdown file (`SKILL.md`) that gives Claude a structured set of instructions, checklists, and guidelines for a specific domain. Skills are activated automatically when relevant topics arise in conversation.

## Directory Structure

```text
skills/
  your-skill-name/
    SKILL.md
```

## SKILL.md Frontmatter

Every skill begins with YAML frontmatter:

```yaml
---
name: your-skill-name
description: ...
disable-model-invocation: false
---
```

Field guidance:

- `name`: unique identifier in kebab-case.
- `description`: explains what the skill does and when it should trigger.
- `disable-model-invocation`: set to `true` only when model invocation should be suppressed.

## Recommended Sections

| Section | Purpose |
|---|---|
| **Purpose** | One-paragraph goal statement |
| **Core Philosophy** | Guiding principles for decisions within this skill |
| **Checklist** | Categorized checks with success criteria |
| **Common Anti-Patterns** | Known failure modes with fixes |
| **Process** | Step-by-step instructions for the model |
| **Output Format** | Expected structure of the model's response |

## Adding a Skill to the Marketplace

After creating your skill, register it in [`.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json) by adding an entry to the `plugins` array:

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
