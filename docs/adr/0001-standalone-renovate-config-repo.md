# 1. Standalone repo for shared Renovate configuration

## Status

Accepted

## Context

Contentful uses [Renovate Bot](https://www.mend.io/renovate/) for automated
dependency updates across many repositories. Renovate supports sharing
configuration between repos via its `local>` preset mechanism, which resolves
presets by fetching another repo directly (see `README.md` and the preset
files at this repo's root: `base.json`, `default.json`,
`defaultNxMonorepo.json`, and the additional named presets). This repo
contains only these JSON preset files, a Backstage `catalog-info.yaml`, and
CircleCI config to validate/publish that descriptor — no application code,
build, or deploy artifact of its own.

There is no recorded discussion captured in this repo of alternatives that
were considered (e.g. keeping shared config inline in a template repo, or
duplicating it per-consumer); the rationale below is inferred from the
repo's structure and the `README.md`, not from a documented decision trail.

## Decision

Keep Renovate preset configuration in its own dedicated repo,
`contentful/renovate-config`, rather than folding it into another repo (such
as a general "templates" or "platform config" repo, or duplicating the
settings into every consuming repo).

## Consequences

- Any repo can adopt shared Renovate policy by adding one `extends` entry
  pointing at this repo, without vendoring or copy-pasting JSON.
- A single merge to this repo's `main` branch changes behavior for every
  consumer on their next Renovate run — there is no version pinning by
  consumers observed in this repo, so changes here should be made carefully
  and reviewed by `@contentful/team-mechagodzilla` (per `.github/CODEOWNERS`).
- The repo carries its own minimal Backstage/CircleCI/TechDocs scaffolding
  (`catalog-info.yaml`, `.circleci/config.yml`, `docs/index.md`) purely to
  satisfy org-wide cataloging and documentation conventions, not because the
  Renovate presets themselves need a build or deploy pipeline.
