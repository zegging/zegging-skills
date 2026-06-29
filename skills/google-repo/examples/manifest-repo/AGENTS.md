# Cross-repo conventions

This manifest assembles three projects into one workspace. Anyone (human or AI agent) working in a workspace synced from this manifest should follow the conventions below.

## Projects

- `service-api` (team-a/service-api) — backend HTTP API.
- `service-web` (team-a/service-web) — frontend consumer of `service-api`.
- `infra-gateway` (team-b/infra-gateway) — reverse proxy fronting both.

## Commit messages

Every commit on a work branch must begin with the task identifier, e.g. `PROJ-1234: ...`. This is how a reviewer connects commits across the three repos.

## Branch naming

Work branches in all three projects share the same name, created via `repo start <name> --all`. Convention: `feature/<task-id>` or `fix/<task-id>`.

## Merge order

When a single task touches multiple projects, merge providers before consumers:

1. `service-api` (provider of new endpoints)
2. `service-web` (consumer of those endpoints)
3. `infra-gateway` (routes traffic to whichever needs to change)

Each MR/PR description must list the sibling MR/PR URLs in the other repos.

## Snapshots

After a multi-repo task is merged, run `repo manifest -r -o snapshots/<label>.xml` and commit it to this manifest repo so the combined state can be reproduced. Use a stable label (release name, date, or task ID).

## Workspaces are disposable

The workspace produced by `repo init && repo sync` is reproducible from this manifest. Do not store state in the workspace that does not exist in one of its three git repos.
