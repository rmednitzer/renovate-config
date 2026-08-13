# CLAUDE.md — `renovate-config`

Publishes the shared Renovate preset for the `rmednitzer` fleet. It is a
**three-file repository** and is meant to stay that way.

Companions: `infra` (owns the actual preset), and every repository that
extends it.

## The one rule

**`default.json5` is a thin re-export. Policy is edited in `infra`.**

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: ["github>rmednitzer/infra:renovate-preset"],
}
```

Only `$schema` and `extends` may appear in that file, and `extends` must be
exactly `["github>rmednitzer/infra:renovate-preset"]`. CI asserts both. If a
task asks for a policy change — a new grouping, a different schedule, an
automerge rule, a manager toggle — the change belongs in
`infra/renovate-preset.json`, not here. Say so rather than editing this file.

### Why the constraint exists

This repository and `infra` both used to carry a full copy of the policy. They
drifted: this copy kept `lockFileMaintenance.automerge: true` after the setting
had been removed from the live preset. On 2026-08-10 a lockfile refresh crossed
a major boundary with no major-version PR and broke `nous` main. The re-export
shape is the fix, and the CI assertion is what keeps the fix from being undone
by a well-meaning edit.

## Layout

```
renovate-config/
├── default.json5                   # THE preset re-export (do not add keys)
├── renovate.json5                  # this repo's OWN update config (different file!)
└── .github/workflows/validate.yml  # the three checks below
```

`default.json5` versus `renovate.json5` is the single most likely confusion:

| File | Read by | Purpose |
|------|---------|---------|
| `default.json5` | *consumers*, as a preset | What the fleet inherits |
| `renovate.json5` | Renovate, for *this* repo | Keeps this repo's own action pins fresh |

Renovate's config discovery covers `renovate.json`, `renovate.json5`,
`.github/renovate.json*`, and `.renovaterc*`. `default.json5` is not in that
list, so it is never mistaken for this repository's own config.

## CI

`validate.yml` runs on every pull request and push to `main`:

1. **Schema and syntax** — `renovate-config-validator --strict`.
2. **Preset resolution** — resolves the `github>` preset over the GitHub API,
   so a renamed or deleted target fails here rather than in a consumer. Needs
   `RENOVATE_GITHUB_COM_TOKEN` (not `RENOVATE_TOKEN`: under `--platform=local`
   the platform is not GitHub, so `RENOVATE_TOKEN` names a platform that is
   never contacted).
3. **Re-export assertion** — fails on any key beyond `$schema` and `extends`,
   or any `extends` target other than the pinned one.

All three run locally; see `CONTRIBUTING.md`.

## Conventions

- GitHub Actions pinned to a **full commit SHA** with a trailing `# vX.Y.Z`
  comment. Renovate keeps them moving.
- Nothing automerges here (`renovate.json5` sets `automerge: false` fleet-wide
  for this repo). This repository's `main` defines dependency policy for every
  consumer, so every change to it should be seen by a human. Update PRs are
  still raised and grouped; only the unattended merge is off.
- Imperative commit subjects. One logical change per commit.

## Notes for AI assistants

- Read `README.md` and `CONTRIBUTING.md` before proposing any change.
- A request to change dependency-update *behaviour* is almost always a request
  to edit `infra/renovate-preset.json`. Redirect rather than adding a key here.
- Never widen `default.json5`. A PR that does will fail CI, and the failure is
  the point.
- Do not commit tokens. The CI token is the ambient `GITHUB_TOKEN`; nothing
  needs to be stored in the repository.
- When changing `validate.yml`, keep `permissions: contents: read` and keep
  every action SHA-pinned.
