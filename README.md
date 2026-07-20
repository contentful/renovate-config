[![Renovate Config Validator](https://github.com/contentful/renovate-config/actions/workflows/renovate-config-validator.yml/badge.svg)](https://github.com/contentful/renovate-config/actions/workflows/renovate-config-validator.yml)

# renovate-config

Configuration presets for renovate in Contentful

## Structure

`base.json` holds the settings shared by every consumer (labels, host rules, vulnerability alerts, release age, docker/npm defaults, and the common `extends` presets). The two entrypoint presets extend it and only add their differences:

- `default.json` — general repos. Extends `base` plus `:preserveSemverRanges` (uses the `replace` range strategy) and `updateTflint`.
- `defaultNxMonorepo.json` — complex Nx monorepos. Extends `base` plus `groupNxPluginAndWorkflows`, and overrides the range strategy to `bump` with per-depType npm rules and terraform scheduling.

When changing a setting that should apply everywhere, edit `base.json` so it stays in one place; only put entrypoint-specific overrides in `default.json` / `defaultNxMonorepo.json`.
