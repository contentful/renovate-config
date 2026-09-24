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

The preset applies only to npm updates for `@contentful/agents-kit` or `@contentful/*skill*`. It performs a full dependency install, runs `contentful-agents-kit skills install` once per Renovate branch, and includes generated changes only from `skills/**`, `.agents/**`, `.claude/**`, and `.cursor/**`.

The Mend-hosted Renovate command is centrally allowed. The consuming repository must declare Agents Kit and its generated skill configuration. Command or install failures surface as Renovate artifact errors.

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
