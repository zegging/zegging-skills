# zegging-skills

A curated marketplace of reusable Claude skills for engineering workflows, code quality, architecture guidance, and domain-specific delivery.

## Available Skills

| Skill | Summary |
|---|---|
| [`fastapi-project-steward`](skills/fastapi-project-steward/SKILL.md) | Guide FastAPI project structure, dependency and service layering, async boundaries, schema design, testing, and maintainable backend architecture work. |
| [`google-repo`](skills/google-repo/SKILL.md) | Coordinate development across multiple independent git repositories using Google's `repo` tool and a manifest, producing reproducible workspaces and version snapshots. |

## Featured — `google-repo`

When you have several independent git repos that must be developed together but should not be collapsed into a monorepo (different ownership, different release cadence, different teams), `google-repo` gives you a versioned manifest that pins exactly which repos sit at which revisions. One `repo init && repo sync` materializes the whole workspace; one manifest commit reproduces it forever.

The skill covers the full lifecycle:

- **Bootstrap** — create the manifest repo, write `default.xml`, initialize the workspace.
- **Daily development** — work branches across N repos, the two-kinds-of-remote trap, cross-repo commit conventions.
- **Review and merge** — one MR/PR per touched repo, cross-referenced, merged in dependency order.
- **Snapshot** — `repo manifest -r` to freeze the combination, tag it, reproduce it from any machine.
- **Manifest evolution** — change the baseline without breaking active work branches.

Each phase ships with a checkable completion criterion. Supporting files cover the XML pitfalls that break `repo sync` (`MANIFEST_REFERENCE.md`), the term overloads that confuse newcomers (`GLOSSARY.md` — including why `git fetch origin` often fails), and a phase-grouped failure index (`TROUBLESHOOTING.md`). A worked example sits at [`skills/google-repo/examples/`](skills/google-repo/examples/).

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
