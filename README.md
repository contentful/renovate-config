[![Renovate Config Validator](https://github.com/contentful/renovate-config/actions/workflows/renovate-config-validator.yml/badge.svg)](https://github.com/contentful/renovate-config/actions/workflows/renovate-config-validator.yml)

# renovate-config

Configuration presets for renovate in Contentful

## Structure

`base.json` holds the settings shared by every consumer (labels, host rules, vulnerability alerts, release age, scheduling, docker/npm defaults, and the common `extends` presets). The two entrypoint presets extend it and only add their differences:

- `default.json` — general repos. Extends `base` plus `:preserveSemverRanges` (uses the `replace` range strategy) and `updateTflint`.
- `defaultNxMonorepo.json` — complex Nx monorepos. Extends `base` plus `groupNxPluginAndWorkflows`, and overrides the range strategy to `bump` with per-depType npm rules and terraform scheduling.
- `syncAgentSkills.json` — optional preset that regenerates repository-shared Agents Kit skills after matching package upgrades.

When changing a setting that should apply everywhere, edit `base.json` so it stays in one place; only put entrypoint-specific overrides in `default.json` / `defaultNxMonorepo.json`.

## Agents Kit skill synchronization

Repositories that commit generated Agents Kit skills can opt in with:

```json
{
  "extends": [
    "local>contentful/renovate-config",
    "local>contentful/renovate-config:syncAgentSkills"
  ]
}
```

The preset groups npm-manager updates for `@contentful/agents-kit` or `@contentful/*skill*`. The generic rule requests installation with `skipInstalls: false`, runs `contentful-agents-kit skills install --allow-destructive` once per Renovate branch, and includes generated changes only from `skills/**`, `.agents/**`, `.claude/**`, and `.cursor/**`.

Skill regeneration is temporarily paused for repositories using a root `pnpm-lock.yaml`, pending [CAO-475](https://contentful.atlassian.net/browse/CAO-475). Renovate's pnpm updater uses `--lockfile-only`, so the local CLI may be unavailable even with `skipInstalls: false`. Dependency updates continue. Refresh committed skills manually on each affected PR branch before merging:

```sh
pnpm install --frozen-lockfile
pnpm exec contentful-agents-kit skills install --allow-destructive
```

After this preset change is merged, request a rebase/retry on failing pnpm PRs and verify that their Renovate artifact status clears.

The flag lets unattended updates replace stale generated skills. It also permits overwriting edits in installed package skills and deleting skills in managed roots that are not declared in the Agents Kit configuration, even if Git tracks them. Keep custom skills registered as directory sources in `agentsKit.skills.uses.sources`; edit those sources rather than generated package copies. Use the default `./skills` shared root because the preset's file filters do not capture custom shared roots.

The Mend-hosted Renovate command is centrally allowed. The consuming repository must declare Agents Kit 0.27.0 or newer, which supports `--allow-destructive`, and its generated skill configuration. Command or install failures surface as Renovate artifact errors.

## Scheduling

The base preset applies [`schedule:nonOfficeHours`](https://docs.renovatebot.com/presets-schedule/#schedulenonofficehours) and [`:noUnscheduledUpdates`](https://docs.renovatebot.com/presets-default/#nounscheduledupdates) to reduce CI disruption during daily engineering work. Routine Renovate branches are created and updated during non-office hours, while security updates retain their separate inherited handling.

The base preset intentionally does not choose a timezone. Each consuming repository should add [`:timezone(<IANA timezone>)`](https://docs.renovatebot.com/presets-default/#timezonearg0) to its own `extends` list using the timezone where most of the team maintaining that repository works. For example:

```json
{
  "extends": [
    "local>contentful/renovate-config:defaultNxMonorepo",
    ":timezone(Europe/Berlin)"
  ]
}
```

Use an IANA timezone such as `Europe/Berlin`, `America/Los_Angeles`, or `Asia/Singapore` so daylight-saving changes are handled automatically where applicable.

`base.json` inherits Renovate's `config:recommended`, which already includes [`group:monorepos`](https://docs.renovatebot.com/presets-group/#groupmonorepos). That built-in grouping covers known upstream monorepos used by Contentful repositories, including Nx, TypeScript-ESLint, Vitest, SWC, AWS SDK for JavaScript, Hapi, GraphQL Tools, Lingui, OpenTelemetry JS, Playwright, Sentry, and Storybook. `defaultNxMonorepo.json` therefore does not repeat those `group:*Monorepo` presets. Its `groupNxPluginAndWorkflows` preset is still needed for Contentful's own Nx and reusable-workflow relationship.

The base preset also uses Renovate's [`group:linters`](https://docs.renovatebot.com/presets-group/#grouplinters) and [`group:jsTest`](https://docs.renovatebot.com/presets-group/#groupjstest) presets. They replace the global ESLint/Prettier, JavaScript test-tooling, and Testing Library rules. `group:jsTest` intentionally covers a wider set of JavaScript test packages, including Jest, Vitest, Testing Library, Mocha, Chai, Sinon, and Nock. Contentful-specific ecosystems, and cross-manager groups such as Playwright's npm packages plus Docker image, remain in `groupCommonPackages.json`.

The existing `groupESLintPrettier` preset remains available for consumers that explicitly extend it, but is no longer applied by `base.json`. Package presets such as [`packages:linters`](https://docs.renovatebot.com/presets-packages/#packageslinters) are matching helpers rather than groups by themselves; use the corresponding `group:*` preset when grouping is desired. [`group:vite`](https://docs.renovatebot.com/presets-group/#groupvite), [`group:react`](https://docs.renovatebot.com/presets-group/#groupreact), and [`group:pnpm`](https://docs.renovatebot.com/presets-group/#grouppnpm) remain opt-in because the repository scan did not show a sufficiently useful global grouping need for them.
