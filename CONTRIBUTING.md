# Contributing

## Proposing a change

Open a PR against `main` from a feature branch. There is no PR template in
this repo. `.github/CODEOWNERS` requires review from
`@contentful/team-mechagodzilla` on every path (`* @contentful/team-mechagodzilla`).

Most changes belong in `base.json` if they should apply to every consumer, or
in the specific entrypoint/preset file (`default.json`,
`defaultNxMonorepo.json`, `node.json`, `terraformCloud.json`,
`groupESLintPrettier.json`, `groupNxPluginAndWorkflows.json`,
`updateTflint.json`) if they're specific to one use case. Avoid duplicating a
setting across files.

## What actually runs in CI

`.circleci/config.yml` defines one workflow, `build`, with three jobs:

- `cf-validations/validation` (named `backstage-validation`) — validates
  `catalog-info.yaml` against Backstage's schema.
- `tech-docs/publish-tech-docs` — publishes the TechDocs site, on `main` only.
- `tech-docs/preview-tech-docs` — previews the TechDocs site, on all
  non-`main` branches.

None of these jobs validate the Renovate preset JSON itself. Each preset file
declares `"$schema": "https://docs.renovatebot.com/renovate-schema.json"`, so
use an editor with JSON schema support to catch structural mistakes before
opening a PR, and double check `extends` references and `matchPackageNames`
patterns by hand.

## After merge

There is no build or release step — merging to `main` is the release.
Consuming repos' Renovate bot runs fetch this repo's `main` branch directly
the next time they run, so a merge here takes effect for all consumers on
their next Renovate run, without any additional promotion step from this
repo.
