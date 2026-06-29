# manifest-repo (example)

Manifest for the `service-api` + `service-web` + `infra-gateway` workspace.

## Bootstrap

```bash
mkdir my-workspace
cd my-workspace
repo init -u https://git.example.com/team-a/manifest-repo.git -b main
repo sync -c -j4
repo start feature/<task-id> --all
```

## Use a frozen release

```bash
repo init -u https://git.example.com/team-a/manifest-repo.git -b main \
          -m snapshots/release/2026.01.01.xml
repo sync -c -j4
```

## Reproduce an incident state

```bash
mkdir incident-repro
cd incident-repro
repo init -u https://git.example.com/team-a/manifest-repo.git -b main \
          -m snapshots/incident/2026-02-14-prod-issue.xml
repo sync -c -j4
```

## Cross-repo rules

See `AGENTS.md`.
