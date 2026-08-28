# ARCHITECTURE.md

This repo provisions nothing at runtime — it is a shared **configuration
library** for [Renovate Bot](https://www.mend.io/renovate/), Contentful's
automated dependency-update tool. It exists as its own repo (rather than
living inside each consuming repo) so that dependency-update policy — labels,
host auth, release-age gates, grouping rules — is defined once and reused by
every repo that wants it, instead of being copy-pasted and drifting per repo.

## How a change here takes effect

Consuming repos point at this repo from their own `renovate.json` using
Renovate's `local>` preset syntax, e.g.:

```json
"extends": ["local>contentful/renovate-config", "local>contentful/renovate-config:node"]
```

There is no build, package, or publish step. When Renovate's GitHub App runs
against a consuming repo, it fetches this repo's default branch directly and
resolves the referenced preset file(s) at that moment. A merge to `main` here
therefore changes behavior for every consumer the next time Renovate runs
against them — there is no versioning or pinning of presets by the consumers
observed in this repo.

## Composition model

`base.json` holds settings common to all consumers. `default.json` and
`defaultNxMonorepo.json` are the two entrypoint presets most repos actually
extend, and each layers only its differences over `base.json`. The remaining
files (`node.json`, `terraformCloud.json`, `groupESLintPrettier.json`,
`groupNxPluginAndWorkflows.json`, `updateTflint.json`) are optional presets a
consumer can add individually to `extends`. This repo has no runtime
component of its own beyond this JSON.

## Supporting mechanisms

`catalog-info.yaml` registers this repo as a Backstage `library` component
(owned by `team-mechagodzilla`); CircleCI (`.circleci/config.yml`) validates
that descriptor and publishes its TechDocs page on merges to `main`. Neither
of these is part of the Renovate delivery path described above.
