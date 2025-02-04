

<!-- markdownlint-disable -->
<a href="https://cpco.io/homepage"><img src="https://github.com/cloudposse/github-action-atmos-affected-stacks/blob/main/.github/banner.png?raw=true" alt="Project Banner"/></a><br/>
    <p align="right">
<a href="https://github.com/cloudposse/github-action-atmos-affected-stacks/releases/latest"><img src="https://img.shields.io/github/release/cloudposse/github-action-atmos-affected-stacks.svg" alt="Latest Release"/></a><a href="https://slack.cloudposse.com"><img src="https://slack.cloudposse.com/badge.svg" alt="Slack Community"/></a></p>
<!-- markdownlint-restore -->

<!--




  ** DO NOT EDIT THIS FILE
  **
  ** This file was automatically generated by the `cloudposse/build-harness`.
  ** 1) Make all changes to `README.yaml`
  ** 2) Run `make init` (you only need to do this once)
  ** 3) Run`make readme` to rebuild this file.
  **
  ** (We maintain HUNDREDS of open source projects. This is how we maintain our sanity.)
  **





-->

A GitHub Action to get a list of affected atmos stacks for a pull request




## Introduction

This is a GitHub Action to get a list of affected atmos stacks for a pull request. It optionally installs 
`atmos` and `jq` and runs `atmos describe affected` to get the list of affected stacks. It provides the 
raw list of affected stacks as an output as well as a matrix that can be used further in GitHub action jobs.




## Usage


### Config

> [!IMPORTANT]
> **Please note!**  This GitHub Action only works with `atmos >= 1.99.0`.
> If you are using `atmos >= 1.80.0, < 1.99.0` please use `v5` version of this action.
> If you are using `atmos >= 1.63.0, < 1.80.0` please use `v3` or `v4` version of this action.
> If you are using `atmos < 1.63.0` please use `v2` version of this action.    


The action expects the atmos configuration file `atmos.yaml` to be present in the repository.

The action supports AWS and Azure to store Terraform plan files. 
You can read more about plan storage in the [cloudposse/github-action-terraform-plan-storage](https://github.com/cloudposse/github-action-terraform-plan-storage?tab=readme-ov-file#aws-default) documentation. 
Depends of cloud provider the following fields should be set in the `atmos.yaml`:

#### AWS

The config should have the following structure:

```yaml
integrations:
  github:
    gitops:
      opentofu-version: 1.7.3  
      terraform-version: 1.5.2
      infracost-enabled: false
      artifact-storage:
        region: us-east-2
        bucket: cptest-core-ue2-auto-gitops
        table: cptest-core-ue2-auto-gitops-plan-storage
        role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
      role:
        plan: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
        # Set `apply` empty if you don't want to assume IAM role before terraform apply
        apply: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
      matrix:
        sort-by: .stack_slug
        group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")
```

#### Azure

The config should have the following structure:

```yaml
integrations:
  github:
    gitops:
      opentofu-version: 1.7.3  
      terraform-version: 1.5.2
      infracost-enabled: false
      artifact-storage:
        plan-repository-type: azureblob
        blob-account-name: tfplans
        blob-container-name: plans
        metadata-repository-type: cosmos
        cosmos-container-name: terraform-plan-storage
        cosmos-database-name: terraform-plan-storage
        cosmos-endpoint: "https://my-cosmo-account.documents.azure.com:443/"
      # We remove the `role` section as it is AWS specific
      matrix:
        sort-by: .stack_slug
        group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")
```

### Stack level configuration

> [!IMPORTANT]
> Wherever it is possible to specify `integration.github.gitops` on stack level 
> it is required to define default values in `atmos.yaml`

It is possible to override integration settings on a stack level by defining `settings.integrations`.

```yaml
components:
  terraform:
    foobar:
      settings:
        integrations:
          github:
            gitops:
              artifact-storage:
                bucket: cptest-plat-ue2-auto-gitops
                table: cptest-plat-ue2-auto-gitops-plan-storage
                role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-plat-ue2-auto-gitops-gha
              role:
                # Set `plan` empty if you don't want to assume IAM role before terraform plan  
                plan: arn:aws:iam::yyyyyyyyyyyy:role/cptest-plat-gbl-identity-gitops
                apply: arn:aws:iam::yyyyyyyyyyyy:role/cptest-plat-gbl-identity-gitops
```    

### Support OpenTofu

This action supports [OpenTofu](https://opentofu.org/).

> [!IMPORTANT]
> **Please note!** OpenTofu supported by Atmos `>= 1.73.0`.
> For details [read](https://atmos.tools/core-concepts/projects/configuration/opentofu/)

To enable OpenTofu add the following settings to `atmos.yaml`
  * Set the `opentofu-version` in the `atmos.yaml` to the desired version
  * Set `components.terraform.command` to `tofu`

#### Example

```yaml

components:
  terraform:
    command: tofu

...

integrations:
  github:
    gitops:
      opentofu-version: 1.7.3
      ...
```  
  
### Workflow example

```yaml
name: Pull Request
on:
  pull_request:
    branches: [ 'main' ]
    types: [opened, synchronize, reopened, closed, labeled, unlabeled]

jobs:
  atmos-affected:
    runs-on: ubuntu-latest
    steps:
      - id: affected
        uses: cloudposse/github-action-atmos-affected-stacks@v6
        with:
          atmos-config-path: ./rootfs/usr/local/etc/atmos/
          atmos-version: 1.99.0
          nested-matrices-count: 1

    outputs:
      matrix: ${{ steps.affected.outputs.matrix }}
      has-affected-stacks: ${{ steps.affected.outputs.has-affected-stacks }}

  # This job is an example how to use the affected stacks with the matrix strategy
  atmos-plan:
    needs: ["atmos-affected"]
    if: ${{ needs.atmos-affected.outputs.has-affected-stacks == 'true' }}
    name: Plan ${{ matrix.stack_slug }}
    runs-on: ubuntu-latest
    strategy:
      max-parallel: 10
      fail-fast: false # Don't fail fast to avoid locking TF State
      matrix: ${{ fromJson(needs.atmos-affected.outputs.matrix) }}
    ## Avoid running the same stack in parallel mode (from different workflows)
    concurrency:
      group: ${{ matrix.stack_slug }}
      cancel-in-progress: false
    steps:
      - name: Plan Atmos Component
        uses: cloudposse/github-action-atmos-terraform-plan@v4
        with:
          component: ${{ matrix.component }}
          stack: ${{ matrix.stack }}
          atmos-config-path: ./rootfs/usr/local/etc/atmos/
          atmos-version: 1.99.0
```

### Migrating from `v5` to `v6`

The notable changes in `v6` are:

- `v6` works only with `atmos >= 1.99.0`
- `v6` allow to skip internal checkout with `skip-checkout` input


The only required migration step is updating atmos version to `>= 1.80.0`


### Migrating from `v4` to `v5`

The notable changes in `v5` are:

- `v5` works only with `atmos >= 1.80.0`
- `v5` supports atmos templating

The only required migration step is updating atmos version to `>= 1.80.0`

### Migrating from `v3` to `v4`

The notable changes in `v4` are:

- `v4` perform aws authentication assuming `integrations.github.gitops.role.plan` IAM role

No special migration steps required
  
### Migrating from `v2` to `v3`

The notable changes in `v3` are:

- `v3` works only with `atmos >= 1.63.0`
- `v3` drops `install-terraform` input because terraform is not required for affected stacks call
- `v3` drops `atmos-gitops-config-path` input and the `./.github/config/atmos-gitops.yaml` config file. Now you have to use GitHub Actions environment variables to specify the location of the `atmos.yaml`.

The following configuration fields now moved to GitHub action inputs with the same names

|         name            |
|-------------------------|
| `atmos-version`         |
| `atmos-config-path`     |


The following configuration fields moved to the `atmos.yaml` configuration file.

|         name             |    YAML path in `atmos.yaml`                    |
|--------------------------|-------------------------------------------------|
| `aws-region`             | `integrations.github.gitops.artifact-storage.region`     | 
| `terraform-state-bucket` | `integrations.github.gitops.artifact-storage.bucket`     |
| `terraform-state-table`  | `integrations.github.gitops.artifact-storage.table`      |
| `terraform-state-role`   | `integrations.github.gitops.artifact-storage.role`       |
| `terraform-plan-role`    | `integrations.github.gitops.role.plan`          |
| `terraform-apply-role`   | `integrations.github.gitops.role.apply`         |
| `terraform-version`      | `integrations.github.gitops.terraform-version`  |
| `enable-infracost`       | `integrations.github.gitops.infracost-enabled`  |
| `sort-by`                | `integrations.github.gitops.matrix.sort-by`     |
| `group-by`               | `integrations.github.gitops.matrix.group-by`    |


For example, to migrate from `v2` to `v3`, you should have something similar to the following in your `atmos.yaml`: 

`./.github/config/atmos.yaml`
```yaml
# ... your existing configuration

integrations:
  github:
    gitops:
      terraform-version: 1.5.2
      infracost-enabled: false
      artifact-storage:
        region: us-east-2
        bucket: cptest-core-ue2-auto-gitops
        table: cptest-core-ue2-auto-gitops-plan-storage
        role: arn:aws:iam::xxxxxxxxxxxx:role/cptest-core-ue2-auto-gitops-gha
      role:
        plan: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
        apply: arn:aws:iam::yyyyyyyyyyyy:role/cptest-core-gbl-identity-gitops
      matrix:
        sort-by: .stack_slug
        group-by: .stack_slug | split("-") | [.[0], .[2]] | join("-")
```

`.github/workflows/main.yaml`
```yaml
  - id: affected
    uses: cloudposse/github-action-atmos-affected-stacks@v3
    with:
      atmos-config-path: ./rootfs/usr/local/etc/atmos/
      atmos-version: 1.63.0
``` 

This corresponds to the `v2` configuration (deprecated) below.

The `v2` configuration file `./.github/config/atmos-gitops.yaml` looked like this:
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

And the `v2` GitHub Action Workflow looked like this.

`.github/workflows/main.yaml`
```yaml
  - id: affected
    uses: cloudposse/github-action-atmos-affected-stacks@v2
    with:
      atmos-gitops-config-path: ./.github/config/atmos-gitops.yaml
```
 
  
### Migrating from `v1` to `v2`

`v2` moves most of the `inputs` to the Atmos GitOps config path `./.github/config/atmos-gitops.yaml`. Simply create this file, transfer your settings to it, then remove the corresponding arguments from your invocations of the `cloudposse/github-action-atmos-affected-stacks` action.

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
      nested-matrices-count: 1
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
| atmos-config-path | The path to the atmos.yaml file | N/A | true |
| atmos-include-dependents | Whether to include dependents of affected stacks in the output | false | false |
| atmos-include-settings | Include the `settings` section for each affected component | false | false |
| atmos-include-spacelift-admin-stacks | Whether to include the Spacelift admin stacks of affected stacks in the output | false | false |
| atmos-pro-base-url | The base URL of Atmos Pro | https://app.cloudposse.com | false |
| atmos-pro-token | The API token to allow Atmos Pro to upload affected stacks |  | false |
| atmos-pro-upload | Whether to upload affected stacks directly to Atmos Pro | false | false |
| atmos-stack | The stack to operate on |  | false |
| atmos-version | The version of atmos to install | >= 1.99.0 | false |
| base-ref | The base ref to checkout. If not provided, the head default branch is used. | N/A | false |
| default-branch | The default branch to use for the base ref. | ${{ github.event.repository.default\_branch }} | false |
| head-ref | The head ref to checkout. If not provided, the head default branch is used. | ${{ github.sha }} | false |
| install-atmos | Whether to install atmos | true | false |
| install-jq | Whether to install jq | false | false |
| jq-force | Whether to force the installation of jq | true | false |
| jq-version | The version of jq to install if install-jq is true | 1.7 | false |
| nested-matrices-count | Number of nested matrices that should be returned as the output (from 1 to 3) | 2 | false |
| skip-atmos-functions | Skip all Atmos functions such as terraform.output | false | false |
| skip-checkout | Disable actions/checkout for head-ref. Useful for when the checkout happens in a previous step and file are modified outside of git through other actions | false | false |


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

For 🐛 bug reports & feature requests, please use the [issue tracker](https://github.com/cloudposse/github-action-atmos-affected-stacks/issues).

In general, PRs are welcome. We follow the typical "fork-and-pull" Git workflow.
 1. Review our [Code of Conduct](https://github.com/cloudposse/github-action-atmos-affected-stacks/?tab=coc-ov-file#code-of-conduct) and [Contributor Guidelines](https://github.com/cloudposse/.github/blob/main/CONTRIBUTING.md).
 2. **Fork** the repo on GitHub
 3. **Clone** the project to your own machine
 4. **Commit** changes to your own branch
 5. **Push** your work back up to your fork
 6. Submit a **Pull Request** so that we can review your changes

**NOTE:** Be sure to merge the latest changes from "upstream" before making a pull request!

### 🌎 Slack Community

Join our [Open Source Community](https://cpco.io/slack?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=slack) on Slack. It's **FREE** for everyone! Our "SweetOps" community is where you get to talk with others who share a similar vision for how to rollout and manage infrastructure. This is the best place to talk shop, ask questions, solicit feedback, and work together as a community to build totally *sweet* infrastructure.

### 📰 Newsletter

Sign up for [our newsletter](https://cpco.io/newsletter?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=newsletter) and join 3,000+ DevOps engineers, CTOs, and founders who get insider access to the latest DevOps trends, so you can always stay in the know.
Dropped straight into your Inbox every week — and usually a 5-minute read.

### 📆 Office Hours <a href="https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=office_hours"><img src="https://img.cloudposse.com/fit-in/200x200/https://cloudposse.com/wp-content/uploads/2019/08/Powered-by-Zoom.png" align="right" /></a>

[Join us every Wednesday via Zoom](https://cloudposse.com/office-hours?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=office_hours) for your weekly dose of insider DevOps trends, AWS news and Terraform insights, all sourced from our SweetOps community, plus a _live Q&A_ that you can’t find anywhere else.
It's **FREE** for everyone!
## License

<a href="https://opensource.org/licenses/Apache-2.0"><img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg?style=for-the-badge" alt="License"></a>

<details>
<summary>Preamble to the Apache License, Version 2.0</summary>
<br/>
<br/>

Complete license is available in the [`LICENSE`](LICENSE) file.

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
</details>

## Trademarks

All other trademarks referenced herein are the property of their respective owners.


---
Copyright © 2017-2025 [Cloud Posse, LLC](https://cpco.io/copyright)


<a href="https://cloudposse.com/readme/footer/link?utm_source=github&utm_medium=readme&utm_campaign=cloudposse/github-action-atmos-affected-stacks&utm_content=readme_footer_link"><img alt="README footer" src="https://cloudposse.com/readme/footer/img"/></a>

<img alt="Beacon" width="0" src="https://ga-beacon.cloudposse.com/UA-76589703-4/cloudposse/github-action-atmos-affected-stacks?pixel&cs=github&cm=readme&an=github-action-atmos-affected-stacks"/>
