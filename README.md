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

The preset groups npm-manager updates for `@contentful/agents-kit` or `@contentful/*skill*` and uses `rangeStrategy: "bump"` for both the CLI and skill packages, including updates within existing semver ranges. Existing schedules, version constraints, and prerelease policies still apply. The generic rule requests installation with `skipInstalls: false`, runs `contentful-agents-kit skills install --allow-destructive` once per Renovate branch, and commits every file the install changes. `skills install` writes only managed output: the `./skills` shared root, native agent folders such as `.agents/skills`, `.claude/skills`, and `.cursor/skills`, [custom install targets](https://github.com/contentful/agents-kit/blob/main/docs/guide/consuming-skills.md#install-into-custom-folders), and `.agents-kit/custom-targets.json`.

For repositories using a root `pnpm-lock.yaml`, a later rule replaces the generic command list with:

```sh
pnpm install --frozen-lockfile --ignore-scripts --ignore-pnpmfile
./node_modules/.bin/contentful-agents-kit skills install --allow-destructive
```

Renovate's pnpm updater uses `--lockfile-only`, so `skipInstalls: false` does not populate `node_modules`. The explicit installation makes the local CLI and configured skill packages available, installs from the updated lockfile without changing it, and skips lifecycle scripts and `.pnpmfile.cjs` execution. The pnpm rule inherits the package group, but replaces the entire `postUpgradeTasks` object, so it explicitly repeats the branch execution mode and generated-file filters. npm and Yarn retain the generic command list.

### Mend-hosted pnpm requirements

The explicit install needs private-registry authentication separately from Renovate's lockfile update. Renovate restores its temporary host-rule `.npmrc` before post-upgrade tasks, so existing `hostRules` do not establish authentication for this install.

Keep the reviewed Agents Kit command and other existing entries, and append these exact, anchored regex strings to the existing JSON array in `RENOVATE_ALLOWED_COMMANDS`:

```text
[
  "^\\./node_modules/\\.bin/contentful-agents-kit skills install --allow-destructive$",
  "^pnpm install --frozen-lockfile --ignore-scripts --ignore-pnpmfile$"
]
```

Keep shell execution disabled and check inherited Mend permissions. These hosted settings are outside this repository.

Store a read token for each required private registry in Mend Credentials. For GitHub Packages, an example `RENOVATE_CUSTOM_ENV_VARIABLES` entry is:

```json
{
  "npm_config_//npm.pkg.github.com/:_authToken": "{{ secrets.GH_PACKAGES_TOKEN }}"
}
```

Merge this entry into existing custom environment settings. `GH_PACKAGES_TOKEN` must refer to a Mend secret with access to the required packages; configure equivalent registry-specific entries if the full dependency installation needs other private registries. Keep tokens in Mend settings, outside repository files and command arguments. The preset cannot grant command permissions or configure hosted credentials itself.

This credential mechanism is documented for pnpm 10, including the `pnpm@10.34.5` repositories in [CAO-475](https://contentful.atlassian.net/browse/CAO-475). Since pnpm 10.34.2, credential placeholders in project/workspace `.npmrc` files are not expanded; direct `npm_config_*` environment settings avoid that restriction. Recheck authentication when changing pnpm major versions. See [pnpm authentication](https://pnpm.io/10.x/npmrc), [Renovate custom environment variables](https://docs.renovatebot.com/self-hosted-configuration/#customenvvariables), and [Mend environment settings](https://docs.renovatebot.com/mend-hosted/environment-variables/#enterprise-and-mend-appsec-users-environment-variables).

The pnpm configuration is a proposal for CAO-475; execution and credential forwarding have not been verified in a Mend-hosted consumer. Full installation fetches repository dependencies and increases job duration. Install and generation failures remain Renovate artifact errors; inspect the installation error first if both commands fail. To restore the temporary pnpm pause, set the later rule's `postUpgradeTasks.commands` to `[]` and refresh skills manually on affected PR branches.

The flag lets unattended updates replace stale generated skills. It also permits overwriting edits in installed package skills and deleting skills in managed roots that are not declared in the Agents Kit configuration, even if Git tracks them. Keep custom skills registered as directory sources in `agentsKit.skills.uses.sources`; edit those sources rather than generated package copies.

The Mend-hosted Renovate command is centrally allowed. The consuming repository must declare Agents Kit 0.27.0 or newer, which supports `--allow-destructive`, and its generated skill configuration. Skill packages that explicitly provide the package root (`"."`) require Agents Kit 1.6.0 or newer. The shared `bump` strategy avoids retaining an outdated locked CLI within an unchanged semver range. Command or install failures surface as Renovate artifact errors.

The pnpm hosted pilot succeeded in [code-review PR #361](https://github.com/contentful/code-review/pull/361), updating Agents Kit to 1.7.3 and the repository bundle to 0.6.9 while committing regenerated skill content, including the CAO-475 branch-pattern correction. The earlier [PR #359](https://github.com/contentful/code-review/pull/359) proved dependency installation but failed during generation with CLI 1.0.0; keep CLI compatibility separate from install and authentication failures.

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
