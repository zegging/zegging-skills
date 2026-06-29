# Glossary

Terms used across `SKILL.md`, `MANIFEST_REFERENCE.md`, and `TROUBLESHOOTING.md`.

## Manifest

A single XML file (`default.xml` by convention) that declares the set of git repositories in a workspace, where each one lives, and which **revision** each one should be checked out at. Lives in its own git repo — the **manifest repository** — so every change to "the canonical combination" is a reviewable commit with history.

## Manifest repository

A git repo whose only contents are manifest XML files (and optionally an `AGENTS.md`, `README.md`, `snapshots/`). Each commit on it represents one **snapshot**. Never put business code here.

## Workspace

A local directory produced by `repo init` + `repo sync`. Contains:

- `.repo/` — `repo` machinery. Holds the local clone of the manifest repo, the bare git repositories of each project, and signing keys.
- One subdirectory per `<project>`, each behaving as a normal git working tree. Its `.git` is a symlink into `.repo/projects/…`.

A workspace is bound to its `.repo/` directory. Moving the workspace breaks the symlinks; copy by re-init'ing instead.

## Project

One git repository as declared by a `<project>` element in the manifest. In the workspace, it lives at the path given by the `path` attribute. Day-to-day development happens here using normal `git` commands.

## Revision

The ref each project should be checked out at, declared by the `revision` attribute on `<project>` (or inherited from `<default>`). Three valid forms:

- `refs/heads/<branch>` — a moving target; `repo sync` fast-forwards to whatever the branch tip is. The default for active development.
- A full 40-char SHA — frozen; `repo sync` always lands on the same commit. The default for snapshots.
- `refs/tags/<tag>` — frozen, named.

Bare branch names (`main`, `master`) without `refs/heads/` work in some `repo` versions but are ambiguous — `repo` may resolve them as tags. Always use the explicit form.

## Snapshot

A point-in-time freeze of "which project at which SHA". Mechanically: a manifest file where every `revision` is a SHA, committed to the manifest repository (usually under `snapshots/`) and tagged. Reproducible forever by `repo init -b <tag>`.

## Detached HEAD

The default state of every project immediately after `repo sync`. The working tree is at the manifest's revision, but no local branch points there. Commits made on detached HEAD are reachable only through the reflog and risk loss on the next `repo sync`. Always `repo start` a work branch before committing.

## Work branch

A local git branch created by `repo start <name>` in one or more projects. Started from the manifest's revision for that project, isolating new work from other branches. Different developers can have differently named work branches on top of the same manifest.

## Two kinds of remote

A workspace contains two distinct concepts that both use the word "remote". They are not the same thing.

| Layer | Defined in | Read by | Purpose |
|---|---|---|---|
| **Manifest remote** — `<remote name="X" fetch="…">` | manifest XML | `repo` itself | Tells `repo` where to clone projects from. The `fetch` URL is concatenated with each project's `name` attribute to form the clone URL. |
| **Git remote** — `[remote "X"]` in `.git/config` | each project's `.git/config` | `git fetch / push` commands | Standard git remote — what `git fetch X` and `git push X` talk to. |

`repo sync` translates the manifest remote into the git remote: it runs `git remote add <name> <url>` inside each project using the manifest remote's `name` attribute. Therefore the git remote is named whatever the manifest remote is named — not `origin`, unless the manifest explicitly named it `origin`.

Consequences:

- `git fetch origin main` fails when the manifest declared the remote as `company`. The correct command is `git fetch company main`.
- Renaming the `<remote>` in the manifest changes every project's git remote name on the next sync.
- If you want both names available locally, add a second git remote manually inside the project — but the `repo`-managed one will be authoritative for future syncs.

## Baseline

Informal term for "what revision a `repo start` should branch off". Equal to the manifest's `revision` for that project at sync time. Changing the baseline is done by editing the manifest, not by `git checkout` in the workspace.

## Reproducibility

The defining promise of `google-repo`: given (manifest URL, manifest ref), anyone can produce the exact same set of project checkouts. Snapshots make this promise concrete by pinning revisions to SHAs. Floating revisions (branches) trade reproducibility for the ability to follow upstream changes day-to-day.
