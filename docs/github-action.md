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
| skip-checkout | Disable actions/checkout for head-ref. Useful for when the checkout happens in a previous step and file are modified outside of git through other actions | false | false |


## Outputs

| Name | Description |
|------|-------------|
| affected | The affected stacks |
| has-affected-stacks | Whether there are affected stacks |
| matrix | The affected stacks as matrix structure suitable for extending matrix size workaround (see README) |
<!-- markdownlint-restore -->
