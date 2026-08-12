# renovate-config

Shared Renovate preset for rmednitzer repositories.

## Where the config actually lives

The preset is maintained in **[`infra/renovate-preset.json`](https://github.com/rmednitzer/infra/blob/main/renovate-preset.json)**.
`default.json5` in this repository is a thin re-export of it, so both entry
points resolve to the same policy and cannot drift apart.

Consume from a repo with either of these; the first is what the fleet uses:

```json5
{ $schema: "https://docs.renovatebot.com/renovate-schema.json", extends: ["github>rmednitzer/infra:renovate-preset"] }
```

```json5
{ $schema: "https://docs.renovatebot.com/renovate-schema.json", extends: ["github>rmednitzer/renovate-config"] }
```

Edit the policy in `infra`, not here.

## Policy

- **Low-risk updates automerge** (minor, patch, digest, pin, pinDigest) once
  required checks pass, via GitHub native auto-merge. The repo needs *Allow
  auto-merge* enabled.
- **A 3-day release-age quarantine gates every automerge.** This is the window
  in which a broken or compromised release is normally yanked, and it is what
  makes unattended merging defensible. `config:best-practices` ships this for
  npm only, so the preset extends it to PyPI, Docker, GitHub Actions, Terraform,
  Galaxy, and pre-commit.
- **Security updates bypass both the weekly window and the quarantine**, so a
  CVE fix is not held for up to a week. A security *major* still gets review.
- **Lockfile maintenance never automerges.** A lockfile refresh crosses a major
  boundary for any dependency whose manifest range has no upper bound, without
  raising a major-version PR. That is how `mcp` 2.0.0 reached `nous` main on
  2026-08-10.
- **All GitHub Actions updates are grouped** into one PR so concurrent bumps do
  not collide on the same workflow files. Majors split into their own
  `github-actions (major)` PR and are not automerged.
- **Majors are separate, labelled `major-review`, and never automerge** — for
  every manager, not just code dependencies.

## Caveat: repos with no requireable status check

GitHub native auto-merge waits on *required* status checks. In a repo where
every PR-triggered workflow carries a `paths` filter, there is no check that is
guaranteed to run, so there is nothing for auto-merge to wait on and it lands
the PR immediately. Such a repo must override `platformAutomerge: false` so
Renovate falls back to evaluating the checks that did report. `ai-stack` is the
worked example.
