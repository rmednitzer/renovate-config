# renovate-config

Shared Renovate preset for rmednitzer repositories.

Consume from a repo by replacing its Renovate config with:

```json5
{ $schema: "https://docs.renovatebot.com/renovate-schema.json", extends: ["github>rmednitzer/renovate-config"] }
```

## Policy
- Low-risk updates (minor, patch, digest, pin) and weekly lockfile maintenance automerge once required checks pass (GitHub native auto-merge; repo must have Allow auto-merge enabled).
- All GitHub Actions updates, including majors, are grouped into one PR to avoid workflow-file conflicts. A group of only minor/patch/digest automerges; a group containing a major waits for review.
- Major updates of code dependencies are separate, labelled major-review, and never automerge.
