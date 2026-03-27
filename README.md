# zegging-skills

A curated marketplace of reusable Claude skills for engineering workflows, code quality, architecture guidance, and domain-specific delivery.

## Repository Layout

```text
.claude-plugin/
  marketplace.json
skills/
  <skill-name>/
    SKILL.md
README.md
```

## How This Repo Works

- Each skill lives under `skills/<skill-name>/`.
- Each skill is defined by a `SKILL.md` file with frontmatter and operating instructions.
- The marketplace manifest lives in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).

## Contributing

When adding or updating a skill:

1. Create or edit the skill under `skills/<skill-name>/`.
2. Keep the skill focused, triggerable, and easy to follow in real usage.
3. Register or update the entry in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
4. Open a PR with the skill change and a short summary of intent.

## Skill Authoring SOP

The previous README guidance has been moved to [docs/skill-authoring-sop.md](docs/skill-authoring-sop.md) so the repository homepage can stay lightweight.

## License

MIT
