# Renovate Config

This configuration contains some default values for [Renovate](https://www.mend.io/renovate/). Renovate serves a very similar purpose to [Dependabot](https://github.blog/2020-06-01-keep-all-your-packages-up-to-date-with-dependabot/), but with some additional capabilities.

## Why Renovate?

You may be interested in adopting Renovate if you use any of the following:

- [CircleCI](https://docs.renovatebot.com/modules/manager/circleci/): Renovate is capable of updating Docker images & Orbs in your CircleCI config
- [Helm charts](https://docs.renovatebot.com/modules/manager/helmv3/): Renovate can update your Helm chart dependencies
- Anything you can write a regex for using the [`regex` manager](https://docs.renovatebot.com/modules/manager/regex/) - see [cf-vault/renovate.json](https://github.com/contentful/cf-vault/blob/main/renovate.json#L9-L23) for an example

You can find a full list of [supported managers here in the Renovate docs](https://docs.renovatebot.com/modules/manager/).

We have also found the [Dependency Dashboard](https://docs.renovatebot.com/key-concepts/dashboard/) (enabled by default using this base configuration, [example here](https://github.com/contentful/cd-scaffolder/issues/2119)) a useful tool for getting an overview of the status of dependency updates.

Another benefit of Renovate is that configuration is shareable/reusable, which means we can be more consistent in how we update dependencies without duplicating the same boilerplate configuration across many repos.

## Setup

If you want to get started with Renovate on your repository, you will need to request for the [GitHub App](https://github.com/organizations/contentful/settings/installations/28065986) to be installed on your repository. You can reach out to [Team Mechagodzilla](https://contentful.roadie.so/catalog/default/group/team-mechagodzilla) if you need any help doing this.

Once the app is installed on your repository, a PR will be created to create the configuration `renovate.json` file. You should merge this to complete the setup of Renovate.

Once you have done this, you can configure any additional managers you may require specifically for your project.
