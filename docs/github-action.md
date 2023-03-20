<!-- markdownlint-disable -->

## Inputs

| Name | Description | Default | Required |
|------|-------------|---------|----------|
| atmos-version | The version of atmos to install if install-atmos is true | latest | false |
| default-branch | The default branch to use for the base ref. | ${{ github.event.repository.default\_branch }} | false |
| head-ref | The head ref to checkout. If not provided, the head default branch is used. | N/A | false |
| install-atmos | Whether to install atmos | true | false |
| install-jq | Whether to install jq | false | false |
| install-terraform | Whether to install terraform | true | false |
| jq-force | Whether to force the installation of jq | true | false |
| jq-version | The version of jq to install if install-jq is true | 1.6 | false |
| terraform-version | The version of terraform to install if install-terraform is true | latest | false |


## Outputs

| Name | Description |
|------|-------------|
| affected | The affected stacks |
| has-affected-stacks | Whether there are affected stacks |
| matrix | A matrix suitable for use for GitHub Actions of the affected stacks |
<!-- markdownlint-restore -->
