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
