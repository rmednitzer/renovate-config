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

## This repository's own updates

Two files, easy to confuse:

| File | Read by | Purpose |
|------|---------|---------|
| `default.json5` | *consumers*, as a preset | The policy the fleet inherits |
| `renovate.json5` | Renovate, for *this* repo | Keeps this repo's own action pins fresh |

Renovate discovers repository config from `renovate.json`, `renovate.json5`,
`.github/renovate.json*`, and `.renovaterc*`. `default.json5` is not in that
list, so it is only ever resolved as a preset by repositories that extend it,
never as this repository's own config. The two cannot shadow each other.

`renovate.json5` exists because `validate.yml` pins its actions to full commit
SHAs. A pinned SHA is correct, but with no manager watching it the pin never
moves and quietly becomes a stale, unpatched version.

**Nothing automerges here**, unlike the fleet default. This repository's `main`
defines dependency policy for every consumer, so the property worth keeping is
that every change to it was seen by a human — which is also what makes the
`infra` F12 record of this repo's exposure ("unreviewed change rather than
unreviewed merge") stay true. Update PRs are still raised, still grouped, and
still gated by the release-age quarantine; only the unattended merge is off.

## Repository documents

| File | Purpose |
|------|---------|
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Where a change belongs, and how to run the three CI checks locally |
| [`SECURITY.md`](SECURITY.md) | Supply-chain threat model and private reporting channel |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) | Contributor Covenant 2.1 |
| [`CLAUDE.md`](CLAUDE.md) | Working notes for AI assistants |
| [`LICENSE`](LICENSE) / [`NOTICE`](NOTICE) | Apache-2.0 |
