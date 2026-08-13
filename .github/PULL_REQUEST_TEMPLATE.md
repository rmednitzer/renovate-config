## Description

<!-- What does this PR change, and why? -->

## Type of change

- [ ] CI or tooling
- [ ] Documentation
- [ ] Re-export target change (rare — explain below why the target moved)
- [ ] Repository configuration

## Checklist

- [ ] `default.json5` still contains only `$schema` and `extends`
- [ ] Policy changes (if any) went to [`infra:renovate-preset`](https://github.com/rmednitzer/infra/blob/main/renovate-preset.json) instead of here
- [ ] CI is green (schema validation, preset resolution, re-export assertion)
- [ ] Any new GitHub Action is pinned to a full commit SHA with a version comment

## Consumer impact

<!--
This repository's output is inherited by every repository that extends it.
State the blast radius: which consumers are affected, and what they will see on
their next Renovate run. Write "none — repository-local change" if that is the
case.
-->
