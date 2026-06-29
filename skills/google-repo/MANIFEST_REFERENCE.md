# Manifest reference

Reference for writing `default.xml` and snapshot files. Complements the official spec at <https://gerrit.googlesource.com/git-repo/+/HEAD/docs/manifest-format.md>; focuses on the parts that bite in practice.

## Minimal template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remote name="origin"
          fetch="<base URL of your git host, with trailing slash>" />

  <default remote="origin"
           sync-j="4" />

  <project name="<group>/<repo-a>"
           path="repo-a"
           revision="refs/heads/main" />

  <project name="<group>/<repo-b>"
           path="repo-b"
           revision="refs/heads/main" />
</manifest>
```

## Elements

### `<remote>`

```xml
<remote name="<id>" fetch="<URL prefix>" />
```

- `name` — identifier referenced by `<project remote="…">` and `<default remote="…">`. **Becomes the name of the git remote** inside each project on disk. See `GLOSSARY.md` → "Two kinds of remote".
- `fetch` — URL prefix. Concatenated with each project's `name` to produce the clone URL. Trailing slash recommended for clarity.
  - `fetch="https://gitlab.example.com/"` + `project name="team/svc"` → `https://gitlab.example.com/team/svc`
  - `fetch="git@gitlab.example.com:"` + `project name="team/svc"` → `git@gitlab.example.com:team/svc` (SSH form)

Multiple `<remote>` elements are allowed; projects pick one via the `remote` attribute.

### `<default>`

```xml
<default remote="<remote-id>"
         revision="refs/heads/main"
         sync-j="4" />
```

Fallbacks for every `<project>` that omits the same attribute. Avoid putting a default `revision` here if projects diverge — it hides which projects are intentionally pinned.

### `<project>`

```xml
<project name="<group>/<repo>"
         path="<subdir>"
         revision="refs/heads/<branch>"
         remote="<remote-id>"           <!-- optional, defaults to <default remote=…> -->
         groups="<group>,<group>" />    <!-- optional -->
```

- `name` — path component on the remote host; concatenated with the `<remote fetch>` URL.
- `path` — subdirectory inside the workspace. Defaults to `name`. Prefer an explicit short value; nested paths work (`backend/svc-a`) but increase symlink fragility on Windows.
- `revision` — see [Revision forms](#revision-forms) below.
- `groups` — comma-separated tags; `repo sync --groups=<set>` filters by them. Useful for "always sync core, sync optional only when asked".

### `<include>`

```xml
<include name="other.xml" />
```

Merges another manifest file into this one (file must be at the manifest-repo root). Useful for splitting a large manifest into themes (`include name="core.xml"` + `include name="optional.xml"`).

## Revision forms

| Form | Example | Behaviour |
|---|---|---|
| Full ref under `refs/heads/` | `refs/heads/main` | Tracks the branch; `repo sync` fast-forwards. **Default for development.** |
| 40-char SHA | `1a2b3c4d…` | Frozen at that commit. **Default for snapshots.** |
| Full tag ref | `refs/tags/v1.2.0` | Frozen at the tag. |
| Bare branch name | `main` | Ambiguous — `repo` may try to resolve as tag first in some versions. **Avoid.** |
| Bare tag name | `v1.2.0` | Ambiguous in the other direction. **Avoid.** |

Always prefer the fully-qualified form (`refs/heads/…` or `refs/tags/…`) or a SHA. The ambiguity around bare names is the single most common source of "manifest looks right but `repo sync` says ref not found".

## XML pitfalls that break `repo sync`

`repo` parses manifests with a strict XML parser. The following all fail loudly and prevent any project from syncing.

### Unescaped `&` in attribute values or text

XML reserves `&` to start an entity. URLs with query parameters (`?a=1&b=2`) are the typical offender. Escape as `&amp;`:

```xml
<!-- WRONG: -->
<!-- See https://example.com/page?id=1&tab=2 -->

<!-- RIGHT: -->
<!-- See https://example.com/page?id=1&amp;tab=2 -->
```

This applies inside comments too — XML comments are still parsed enough to enforce some rules.

### `--` inside a comment

XML comments may not contain `--` for any reason — not as a separator, not inside a command-line flag. The parser treats `--` as starting a comment-end.

```xml
<!-- WRONG: -->
<!-- run: repo sync -c --no-clone-bundle -->

<!-- RIGHT (rewrite to avoid the double dash): -->
<!-- run: repo sync (with -c and no-clone-bundle flags) -->
```

### Tag mismatch / unclosed element

`<project … />` (self-closing) and `<project …></project>` (paired) both work. `<project …>` alone does not. Easy to introduce by accident when adding child elements then deleting them.

### Always validate before pushing

Any of these:

```bash
xmllint --noout default.xml
python -c 'import xml.etree.ElementTree as E; E.parse("default.xml")'
```

A 1-second local check beats discovering the parse error from N developers' failed syncs.

## Recovery after pushing a broken manifest

If a manifest with a parse error reaches the remote, every fresh `repo sync` fails before it can fetch the *next* manifest commit that fixes it. The workspace is stuck reading its locally-cached broken copy.

Fix in two steps:

1. Push a corrected manifest commit (the fix).
2. In each workspace, manually update the local manifest clone:

   ```bash
   git -C .repo/manifests fetch origin <manifest-branch>
   git -C .repo/manifests reset --hard origin/<manifest-branch>
   repo sync -c
   ```

After this, future broken-manifest pushes can be recovered the same way. The mistake to learn from: validate XML *before* every push to the manifest repo.

## Snapshot files

Generated by `repo manifest -r -o snapshots/<name>.xml`. Structurally identical to `default.xml` but every `revision` is a 40-char SHA. Treat snapshots as append-only history — name them with a stable identifier (release tag, date, task ID) and never rewrite an existing one. To "amend" a snapshot, write a new file and update the tag.

Recommended layout inside the manifest repo:

```text
manifest-repo/
├── default.xml                       # the live, branch-tracking manifest
├── snapshots/
│   ├── 2026-06-29-task-LLZZ-4770.xml
│   ├── release/
│   │   ├── 2026.07.01.xml
│   │   └── 2026.07.08.xml
│   └── incident/
│       └── 2026-07-03-prod-issue.xml
├── AGENTS.md                         # cross-repo conventions
└── README.md                         # how to use this workspace
```

## Multiple manifests in one repo

Pass `-m <file>` to `repo init` to pick a non-default manifest:

```bash
repo init -u <manifest-URL> -b main -m snapshots/release/2026.07.01.xml
```

Useful when one manifest repo serves multiple consumers (development, CI, release branches).
