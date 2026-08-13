# Security Policy

## What this repository is

`renovate-config` publishes a Renovate preset. It contains no application code
and no runtime. Its security relevance is entirely **supply-chain**: any
repository that extends `github>rmednitzer/renovate-config` inherits the
dependency-update policy this repository resolves to, and a change here
propagates to every consumer on their next Renovate run.

Because `default.json5` is a thin re-export of
[`infra:renovate-preset`](https://github.com/rmednitzer/infra/blob/main/renovate-preset.json),
the effective policy is defined in `infra`. A vulnerability report about
*policy content* (for example, an automerge rule that is too permissive)
belongs in `infra`. A report about *this* repository covers the re-export
itself, the CI workflow, and the repository's own configuration.

## Supported versions

Only the current `main` is supported. Renovate resolves presets from the
default branch, so there are no released versions to patch and no backports.

| Version | Supported |
| ------- | --------- |
| `main`  | Yes       |
| Any tag or older commit | No |

## Reporting a vulnerability

Report privately through GitHub Security Advisories:

**https://github.com/rmednitzer/renovate-config/security/advisories/new**

Please do not open a public issue for a security report. Include the affected
file or workflow, the impact you believe it has on consuming repositories, and
a reproduction or a proof of the propagation path where possible.

Expect an acknowledgement within 7 days. Fixes land on `main` directly, since
that is the only branch consumers resolve.

## Threat model

The realistic threats to this repository, in rough order of severity:

1. **Unreviewed change to `main`.** An attacker who can push to `main` changes
   fleet-wide dependency policy without review. Mitigated by branch protection
   on `main` and by the `Assert the file stays a thin re-export` CI step, which
   fails on any key other than `$schema` and `extends`.
2. **Preset redirection.** Repointing `extends` at an attacker-controlled
   preset would hand them the policy for every consumer. The same CI step pins
   the expected value exactly and fails on any other target.
3. **Upstream drift.** If `infra:renovate-preset` is renamed, moved, or
   deleted, consumers get a broken Renovate run rather than a silent policy
   change. The `Preset resolution check` CI step resolves the preset over the
   GitHub API on every pull request so this surfaces here, not in a consumer.
4. **Workflow compromise.** The CI workflow runs `npx` against the network.
   It holds `contents: read` only, and every action is pinned to a full commit
   SHA.

## What is out of scope

- Vulnerabilities in Renovate itself. Report those to
  [renovatebot/renovate](https://github.com/renovatebot/renovate/security).
- The policy decisions in `infra:renovate-preset`. Report those against
  [`rmednitzer/infra`](https://github.com/rmednitzer/infra/security/advisories/new).
- Vulnerabilities in the dependencies of consuming repositories. Those are what
  the preset exists to surface, not defects in the preset.
