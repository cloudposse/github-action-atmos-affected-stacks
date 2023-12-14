
<!-- markdownlint-disable -->
# github-action-atmos-affected [![Latest Release](https://img.shields.io/github/release/cloudposse/github-action-atmos-affected-stacks.svg)](https://github.com/cloudposse/github-action-atmos-affected-stacks/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)
<!-- markdownlint-restore -->

[![README Header][readme_header_img]][readme_header_link]

[![Cloud Posse][logo]](https://cpco.io/homepage)

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

A GitHub Action to get a list of affected atmos stacks for a pull request

---

This project is part of our comprehensive ["SweetOps"](https://cpco.io/sweetops) approach towards DevOps.


It's 100% Open Source and licensed under the [APACHE2](LICENSE).












## Introduction

This is a GitHub Action to get a list of affected atmos stacks for a pull request. It optionally installs 
`atmos`, `terraform` and `jq` and runs `atmos describe affected` to get the list of affected stacks. It provides the 
raw list of affected stacks as an output as well as a matrix that can be used further in GitHub action jobs.



## Usage




### Config

The action expects the atmos gitops configuration file to be present in the repository in `./.github/config/atmos-gitops.yaml`.
The config should have the following structure:

```yaml
  atmos-version: 1.45.3
  atmos-config-path: ./rootfs/usr/local/etc/atmos/
  terraform-state-bucket: cptest-core-ue2-auto-gitops
  terraform-state-table: cptest-core-ue2-auto-gitops
  terraform-state-role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
  terraform-plan-role: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
  terraform-apply-role: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
  terraform-version: 1.5.2
  aws-region: us-east-2
  enable-infracost: false
  sort-by: .stack_slug
  group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")  
```

### Workflow example

```yaml
  name: Pull Request
  on:
    pull_request:
      branches: [ 'main' ]
      types: [opened, synchronize, reopened, closed, labeled, unlabeled]

  jobs:
    context:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - id: affected
          uses: cloudposse/github-action-atmos-affected-stacks@v2
          with:
            atmos-gitops-config-path: ./.github/config/atmos-gitops.yaml
            nested-matrices-count: 1

      outputs:
        affected: ${{ steps.affected.outputs.affected }}
        matrix: ${{ steps.affected.outputs.matrix }}

    atmos-plan:
      needs: ["atmos-affected"]
      if: ${{ needs.atmos-affected.outputs.has-affected-stacks }}
      name: ${{ matrix.stack_slug }}
      runs-on: ['self-hosted']
      strategy:
        max-parallel: 10
        fail-fast: false # Don't fail fast to avoid locking TF State
        matrix: ${{ fromJson(needs.atmos-affected.outputs.stacks) }}
      ## Avoid running the same stack in parallel mode (from different workflows)
      concurrency:
        group: ${{ matrix.stack_slug }}
        cancel-in-progress: false
      steps:
        - name: Plan Atmos Component
          uses: cloudposse/github-action-atmos-terraform-plan@v1
          with:
            component: ${{ matrix.component }}
            stack: ${{ matrix.stack }}
```
  
### Migrating from `v1` to `v2`

1. `v2` moves most of the `inputs` to the Atmos GitOps config path `./.github/config/atmos-gitops.yaml`. Simply create this file, transfer your settings to it, then remove the corresponding arguments from your invocations of the `cloudposse/github-action-atmos-terraform-apply` action.

|         name             |
|--------------------------|
| `atmos-version`          |
| `atmos-config-path`      |
| `terraform-state-bucket` |
| `terraform-state-table`  |
| `terraform-state-role`   |
| `terraform-plan-role`    |
| `terraform-apply-role`   |
| `terraform-version`      |
| `aws-region`             |
| `enable-infracost`       |


If you want the same behavior in `v2` as in `v1` you should create config `./.github/config/atmos-gitops.yaml` with the same variables as in `v1` inputs.

```yaml
  - name: Determine Affected Stacks
    uses: cloudposse/github-action-atmos-affected-stacks@v2
    id: affected
    with:
      atmos-gitops-config-path: ./.github/config/atmos-gitops.yaml
```

Which would produce the same behavior as in `v1`, doing this:

```yaml
  - name: Determine Affected Stacks
    uses: cloudposse/github-action-atmos-affected-stacks@v1
    id: affected
    with:
      atmos-version: 1.45.3
      atmos-config-path: ./rootfs/usr/local/etc/atmos/
      terraform-state-bucket: cptest-core-ue2-auto-gitops
      terraform-state-table: cptest-core-ue2-auto-gitops
      terraform-state-role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
      terraform-plan-role: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
      terraform-apply-role: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
      terraform-version: 1.5.2
      aws-region: us-east-2
      enable-infracost: false
```






<!-- markdownlint-disable -->

## Inputs

| Name | Description | Default | Required |
|------|-------------|---------|----------|
| atmos-gitops-config-path | The path to the atmos-gitops.yaml file | ./.github/config/atmos-gitops.yaml | false |
| atmos-include-spacelift-admin-stacks | Whether to include the Spacelift admin stacks of affected stacks in the output | false | false |
| base-ref | The base ref to checkout. If not provided, the head default branch is used. | N/A | false |
| default-branch | The default branch to use for the base ref. | ${{ github.event.repository.default\_branch }} | false |
| head-ref | The head ref to checkout. If not provided, the head default branch is used. | ${{ github.sha }} | false |
| install-atmos | Whether to install atmos | true | false |
| install-jq | Whether to install jq | false | false |
| install-terraform | Whether to install terraform | true | false |
| jq-force | Whether to force the installation of jq | true | false |
| jq-version | The version of jq to install if install-jq is true | 1.6 | false |
| nested-matrices-count | Number of nested matrices that should be returned as the output (from 1 to 3) | 2 | false |


## Outputs

| Name | Description |
|------|-------------|
| affected | The affected stacks |
| has-affected-stacks | Whether there are affected stacks |
| matrix | The affected stacks as matrix structure suitable for extending matrix size workaround (see README) |
<!-- markdownlint-restore -->


## Related Projects

Check out these related projects.



## References

For additional context, refer to some of these links.



## ✨ Contributing

This project is under active development, and we encourage contributions from our community. 
Many thanks to our outstanding contributors:

<a href="https://github.com/cloudposse/github-action-atmos-affected-stacks/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=cloudposse/github-action-atmos-affected-stacks&max=24" />
</a>

### 🐛 Bug Reports & Feature Requests

Please use the [issue tracker](https://github.com/cloudposse/github-action-atmos-affected-stacks/issues) to report any bugs or file feature requests.

### 💻 Developing

If you are interested in being a contributor and want to get involved in developing this project or [help out](https://cpco.io/help-out) with our other projects, we would love to hear from you! Shoot us an [email][email].

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.

 1. **Fork** the repo on GitHub
 2. **Clone** the project to your own machine
 3. **Commit** changes to your own branch
 4. **Push** your work back up to your fork
 5. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

### 🌎 Slack Community

Join our [Open Source Community][slack] on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

### 📰 Newsletter

Sign up for [our newsletter][newsletter] that covers everything on our technology radar.  Receive updates on what we're up to on GitHub as well as awesome new projects we discover.

### 📆 Office Hours <img src="https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png" align="right" />

[Join us every Wednesday via Zoom][office_hours] for our weekly "Lunch & Learn" sessions. It's **FREE** for everyone!

## About 

This project is maintained and funded by [Cloud Posse, LLC][website]. 
<a href="https://cpco.io/homepage"><img src="https://cloudposse.com/logo-300x69.svg" align="right" /></a>

We are a [**DevOps Accelerator**][commercial_support]. We'll help you build your cloud infrastructure from the ground up so you can own it. Then we'll show you how to operate it and stick around for as long as you need us.

[![Learn More](https://img.shields.io/badge/learn%20more-success.svg?style=for-the-badge)][commercial_support]

Work directly with our team of DevOps experts via email, slack, and video conferencing.

We deliver 10x the value for a fraction of the cost of a full-time engineer. Our track record is not even funny. If you want things done right and you need it done FAST, then we're your best bet.

- **Reference Architecture.** You'll get everything you need from the ground up built using 100% infrastructure as code.
- **Release Engineering.** You'll have end-to-end CI/CD with unlimited staging environments.
- **Site Reliability Engineering.** You'll have total visibility into your apps and microservices.
- **Security Baseline.** You'll have built-in governance with accountability and audit logs for all changes.
- **GitOps.** You'll be able to operate your infrastructure via Pull Requests.
- **Training.** You'll receive hands-on training so your team can operate what we build.
- **Questions.** You'll have a direct line of communication between our teams via a Shared Slack channel.
- **Troubleshooting.** You'll get help to triage when things aren't working.
- **Code Reviews.** You'll receive constructive feedback on Pull Requests.
- **Bug Fixes.** We'll rapidly work with you to fix any bugs in our projects.

[![README Commercial Support][readme_commercial_support_img]][readme_commercial_support_link]
## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

```text
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```

## Trademarks

All other trademarks referenced herein are the property of their respective owners.
---
Copyright © 2017-2023 [Cloud Posse, LLC](https://cpco.io/copyright)
[![README Footer][readme_footer_img]][readme_footer_link]
[![Beacon][beacon]][website]
<!-- markdownlint-disable -->
  [logo]: https://cloudposse.com/logo-300x69.svg
  [docs]: https://cpco.io/docs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=docs
  [website]: https://cpco.io/homepage?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=website
  [github]: https://cpco.io/github?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=github
  [jobs]: https://cpco.io/jobs?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=jobs
  [hire]: https://cpco.io/hire?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=hire
  [slack]: https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=slack
  [twitter]: https://cpco.io/twitter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=twitter
  [office_hours]: https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=office_hours
  [newsletter]: https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=newsletter
  [email]: https://cpco.io/email?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=email
  [commercial_support]: https://cpco.io/commercial-support?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=commercial_support
  [we_love_open_source]: https://cpco.io/we-love-open-source?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=we_love_open_source
  [terraform_modules]: https://cpco.io/terraform-modules?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=terraform_modules
  [readme_header_img]: https://cloudposse.com/readme/header/img
  [readme_header_link]: https://cloudposse.com/readme/header/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=readme_header_link
  [readme_footer_img]: https://cloudposse.com/readme/footer/img
  [readme_footer_link]: https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=readme_footer_link
  [readme_commercial_support_img]: https://cloudposse.com/readme/commercial-support/img
  [readme_commercial_support_link]: https://cloudposse.com/readme/commercial-support/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=readme_commercial_support_link
  [beacon]: https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/github-action-atmos-affected-stacks?pixel&cs=github&cm=readme&an=github-action-atmos-affected-stacks
<!-- markdownlint-restore -->
