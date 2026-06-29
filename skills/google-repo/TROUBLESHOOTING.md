# Troubleshooting

Grouped by the phase in which the symptom appears. Each entry: **symptom → cause → fix**.

## During `repo init`

### `Cannot get https://gerrit.googlesource.com/git-repo/clone.bundle`

**Cause.** `repo init` always clones its own implementation source from `gerrit.googlesource.com` into `.repo/repo/`. If that host is unreachable (firewall, network restriction, no proxy), init fails before it ever touches the manifest.

**Fix.** Make `gerrit.googlesource.com` reachable to `git` and Python. Typical levers, in order:

1. Set `HTTP_PROXY` / `HTTPS_PROXY` if a proxy is required for external traffic.
2. Configure git's proxy: `git config --global http.proxy <url>`.
3. Add a `NO_PROXY` exception for internal git hosts so the proxy is *only* used for external hosts. The manifest repo and the projects almost always live on a different (internal) host than `gerrit.googlesource.com`; without `NO_PROXY`, internal fetches break.

### `PermissionError: client does not have the required privilege` on Windows

**Cause.** `repo` represents each project's `.git` as a symlink into `.repo/projects/`. Windows blocks symlink creation for non-admin users by default.

**Fix.** Enable Developer Mode (Settings → Privacy & security → For developers → Developer Mode). No reboot needed. Remove the partial `.repo/` and re-run `repo init`.

### `gpg (GnuPG) is not available` warning

**Cause.** `repo` would verify a signed manifest tag if gpg existed. Without gpg, signature verification is skipped.

**Fix.** Harmless — ignore. Install gpg if signature verification is a real requirement.

## During `repo sync`

### `error parsing manifest …: not well-formed (invalid token): line N, column M`

**Cause.** The local cached manifest in `.repo/manifests/` has invalid XML. Usually an unescaped `&` or `--` inside a comment. See `MANIFEST_REFERENCE.md` → "XML pitfalls".

**Fix.** Push a corrected manifest to the remote, then *manually update the local cache* — because `repo sync` cannot fetch the new manifest while the cached one is unparseable:

```bash
git -C .repo/manifests fetch origin <manifest-branch>
git -C .repo/manifests reset --hard origin/<manifest-branch>
repo sync -c
```

### `couldn't find remote ref refs/heads/<branch>`

**Cause.** The manifest declares a `revision` that does not exist on the remote of the listed project. Common reasons: typo in branch name, branch since deleted, or using a bare branch name that `repo` resolved as a tag.

**Fix.** Verify the ref exists: `git ls-remote <project-URL> | grep <name>`. Update the manifest to a ref that actually exists; prefer the fully-qualified `refs/heads/<branch>` form.

### `OpenSSL SSL_connect: SSL_ERROR_SYSCALL` / `Could not resolve host` on internal hosts

**Cause.** A proxy is intercepting traffic to a host that should be reached directly (internal git host being routed through an external proxy).

**Fix.** Add the internal host to `NO_PROXY`. The variable accepts a comma-separated list, and `.example.com` (leading dot) matches the domain and its subdomains in most implementations:

```bash
export NO_PROXY="git.internal.example.com,.internal.example.com,localhost,127.0.0.1"
```

### `<project>/: contains uncommitted changes`

**Cause.** `repo sync` refuses to update a project's working tree while local changes are uncommitted, to avoid destroying work.

**Fix.** Inside that project: stash (`git stash`) or commit, then sync, then unstash.

### Sync hangs on a single project

**Cause.** Usually a slow initial clone of a large repo. Less commonly, a server-side stall.

**Fix.** Re-run with `-j1 --verbose` to isolate which project and see live progress. Add `--no-clone-bundle` if the remote does not serve clone bundles efficiently.

### Rebase fails during sync

**Cause.** A work branch created by `repo start` cannot be cleanly fast-forwarded onto the new revision because of conflicting commits.

**Fix.** `repo sync` reports the failing projects and continues with others. For each failure:

```bash
cd <project>
git status                       # inspect conflicts
# resolve conflicts in working tree
git add <files>
git rebase --continue
```

For massively diverged branches, `git reset --hard <new-baseline>` + cherry-pick is often faster than rebasing.

## During daily development

### `HEAD detached at <sha>` when you expected a branch

**Cause.** You ran `repo sync` (or never ran `repo start`). `repo` deliberately leaves projects detached after sync.

**Fix.** `repo start <branch> <project>` (or `--all`) to create and switch to a work branch off the manifest's revision.

### Commits made on detached HEAD seem lost

**Cause.** A subsequent `repo sync` moved HEAD to a new revision, leaving the commits unreachable through any branch — but still reachable via reflog.

**Fix.**

```bash
cd <project>
git reflog                       # find the commit SHA
git branch <rescue-branch> <sha> # name it before it expires
git checkout <intended-work-branch>
git merge <rescue-branch>
```

Reflog entries expire (default 90 days), so rescue promptly.

### `git fetch origin` fails with "no such remote"

**Cause.** The project's git remote is *not* named `origin` — it is named whatever the manifest's `<remote name="…">` says. See `GLOSSARY.md` → "Two kinds of remote".

**Fix.** Use the correct name: `git remote -v` to see it, then `git fetch <that-name> <branch>`.

### `repo branches` shows a project missing the work branch

**Cause.** `repo start <branch> --all` was run before that project existed in the workspace, or you ran `repo abandon` for it.

**Fix.** Re-run `repo start <branch> <project>` for the missing one.

## During manifest editing

### Pushed a manifest change but workspaces still see the old one

**Cause.** `repo sync` does fetch the manifest first — but if your workspace also has local commits on `.repo/manifests/` (manual edits), the fetch fast-forward fails silently and the old manifest is still in effect.

**Fix.** Never edit `.repo/manifests/` directly. If you already did:

```bash
git -C .repo/manifests reset --hard origin/<manifest-branch>
repo sync -c
```

### Renamed a project's `path`, old directory still present

**Cause.** `repo sync` creates the new path but does not delete the old one (avoids destroying potentially uncommitted work).

**Fix.** Manually delete the old directory in each workspace after confirming nothing is uncommitted:

```bash
rm -rf <old-path>
```

### Removed a project from the manifest, its directory still present

**Cause.** Same as above — `repo` leaves orphans for safety.

**Fix.** Manual `rm -rf <project-path>` after confirming no uncommitted work.

## Workspace corruption

### `.repo/` is damaged (cryptic git errors, missing refs)

**Cause.** Rare. Killed `repo sync` mid-init, filesystem error, virus scanner deleting a `.git` symlink target, etc.

**Fix.** Re-init is cheap because business code lives in working trees, not in `.repo/`:

1. In each project, commit or copy out anything uncommitted *first* (the project's `.git` lives inside `.repo/`, so stashing won't survive the next step).
2. `rm -rf .repo/`
3. `repo init -u <manifest-URL> -b <branch>`
4. `repo sync -c`
5. Restore uncommitted work in each project, recreate work branches with `repo start`.

### Workspace moved to a new path, everything broken

**Cause.** Each project's `.git` is a symlink into `.repo/projects/` using paths anchored at the original location. Moving the workspace breaks them.

**Fix.** Workspaces are not movable. Re-init in the new location. If the old workspace had unpushed commits, push them from the old location first, or copy out the affected `<project>/` working trees and replay the commits manually after re-init.
