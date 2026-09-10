# AGENTS.md

This repo is a library of shared [Renovate Bot](https://www.mend.io/renovate/)
config presets, consumed by other Contentful repos via
`local>contentful/renovate-config[:presetName]` in their `renovate.json`. It
does not build, run, or deploy anything itself — it only publishes JSON.

## Where things live

- `base.json` — settings shared by every consumer (labels, host rules,
  vulnerability alerts, release age, non-office-hours scheduling, and
  docker/npm defaults). Edit here when a change should apply to all consumers.
  Timezone selection stays with each consuming repository because it depends on
  where that repository's maintaining team works.
- `default.json` / `defaultNxMonorepo.json` — the two entrypoint presets most
  repos actually extend; each layers only its differences on top of `base`.
- `node.json`, `terraformCloud.json`, `groupESLintPrettier.json`,
  `groupNxPluginAndWorkflows.json`, `updateTflint.json` — additional named
  presets a consumer can opt into individually.
- `catalog-info.yaml` — Backstage component descriptor for this repo, unrelated
  to the Renovate presets themselves.

## Making a change

Every preset file starts with
`"$schema": "https://docs.renovatebot.com/renovate-schema.json"` — most
editors/IDEs will validate structure and known keys against that schema as you
type. There is no local build or test command in this repo (no
`package.json`, no Makefile): the real correctness check only happens when a
consuming repo's Renovate bot run picks up the merged config. Keep changes
small and prefer editing `base.json` over duplicating a setting into both
entrypoint presets.

## CI you'll see on a PR

`.circleci/config.yml` runs Backstage validation of `catalog-info.yaml`
(`cf-validations/validation`) and publishes/previews TechDocs
(`tech-docs/publish-tech-docs` on `main`, `tech-docs/preview-tech-docs`
otherwise). Neither of these checks the Renovate JSON semantics — there is no
CI job that lints the preset files against the Renovate schema, so review
JSON changes carefully by hand.

## Trip-ups specific to this repo

- The README's CI badge links to a `renovate-config-validator.yml` GitHub
  Actions workflow that does not exist in this checkout — the actual CI is
  CircleCI, not GitHub Actions. Don't assume that workflow file exists.
- Presets reference each other by filename stem (e.g. `:node`, `:base`) via
  the `local>contentful/renovate-config:name` syntax — renaming a file breaks
  every consumer that extends it by that name.
