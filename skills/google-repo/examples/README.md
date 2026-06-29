# Examples

A worked example of the file shapes the `google-repo` skill describes. The directory `manifest-repo/` is meant to be read as if it were a freshly created git repository — copy its layout to your own manifest repo and replace the placeholder URLs and project names.

```text
manifest-repo/
├── default.xml                       # the live manifest tracked during development
├── snapshots/
│   ├── release/
│   │   └── 2026.01.01.xml            # a frozen release combination
│   └── incident/
│       └── 2026-02-14-prod-issue.xml # state captured during an incident
├── AGENTS.md                         # cross-repo conventions for human + AI contributors
└── README.md                         # how to bootstrap a workspace from this manifest
```

Replace these placeholders before using:

- `git.example.com` → your git host.
- `team-a`, `team-b` → real groups/namespaces.
- `service-api`, `service-web`, `infra-gateway` → real project paths on the host.

The two snapshot files use synthetic 40-character SHAs (`a` and `b` repeated) so the schema is realistic but obviously fake.

For the rules each file follows, see `MANIFEST_REFERENCE.md` in the parent directory.
