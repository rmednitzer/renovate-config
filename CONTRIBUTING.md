# Contributing to `renovate-config`

Thanks for looking. This repository is unusual: it is deliberately almost
empty, and the most useful contribution is often a change made *somewhere
else*.

## Read this first: policy is not edited here

`default.json5` is a **thin re-export** of
[`infra:renovate-preset`](https://github.com/rmednitzer/infra/blob/main/renovate-preset.json).
It exists so that `github>rmednitzer/renovate-config` and
`github>rmednitzer/infra:renovate-preset` resolve to the same policy and cannot
drift apart.

They *did* drift once. This repository carried
`lockFileMaintenance.automerge: true` after that setting had been removed from
the live preset, and a lockfile refresh crossed a major boundary and broke
`nous` main on 2026-08-10. The re-export shape, and the CI check that enforces
it, are the fix.

So:

| You want to change | Open the PR against |
| --- | --- |
| An automerge rule, a grouping, a schedule, a release-age window, a manager | [`rmednitzer/infra`](https://github.com/rmednitzer/infra) → `renovate-preset.json` |
| The re-export target, the CI workflow, or this repository's own docs | here |

A pull request that adds a policy key to `default.json5` will fail CI by
design. The `Assert the file stays a thin re-export` step permits `$schema` and
`extends` and nothing else.

## Making a change here

1. Branch from `main` with a descriptive name (`fix/preset-target`,
   `ci/pin-setup-node`, `docs/clarify-automerge-caveat`).
2. Make the change.
3. Push and open a pull request. CI must be green before merge.

### Running the checks locally

The CI workflow is three steps and all of them run locally:

```sh
# 1. Schema and syntax
npx --yes --package renovate -- renovate-config-validator --no-global --strict default.json5

# 2. Preset resolution (needs a token; any token that can read a public repo)
RENOVATE_CONFIG_FILE=default.json5 \
RENOVATE_GITHUB_COM_TOKEN="$GITHUB_TOKEN" \
  npx --yes --package renovate -- renovate --platform=local --dry-run=extract

# 3. The re-export assertion
npx --yes json5 default.json5 -o /tmp/default.json
jq -c '.extends, (keys - ["$schema","extends"])' /tmp/default.json
```

Step 2 needs `RENOVATE_GITHUB_COM_TOKEN`, not `RENOVATE_TOKEN`. Under
`--platform=local` the platform is not GitHub, so `RENOVATE_TOKEN` names a
platform that is never contacted; `github.com` preset fetches read the
`github.com` host rule instead.

## Commit and PR conventions

- Imperative commit subjects (`Pin setup-node to v7`, not `pinned setup-node`).
- One logical change per commit.
- Explain *why* in the body when the change is not self-evident. This
  repository's whole point is a constraint that is easy to undo by accident,
  so reasoning is worth writing down.

## Reporting problems

- A broken Renovate run in a consuming repository: open an issue here with the
  consumer's run log, even if the root cause turns out to be in `infra`.
- A security concern: see [`SECURITY.md`](SECURITY.md). Do not open a public
  issue.

## Code of conduct

Participation is governed by [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).
