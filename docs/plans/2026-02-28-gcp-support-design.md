# GCP Support Design: Documentation-Only Approach

## Date: 2026-02-28

## Summary

Add GCP support to `github-action-atmos-affected-stacks` via documentation only,
following the same pattern used for Azure support (pre-authentication by the user).

## Background

The action currently has hard-coded AWS authentication (`aws-actions/configure-aws-credentials@v4`)
that fires conditionally when `aws-region` and `terraform-plan-role` are present in the atmos config.
Azure support works passively by omitting these AWS-specific keys, causing the AWS auth step to skip.
GCP follows the same pattern.

The downstream `cloudposse/github-action-terraform-plan-storage` action already supports GCP
(`planRepositoryType: gcs`, `metadataRepositoryType: firestore`), so the storage layer is ready.

## Decision

**Documentation-only** (no code changes to `action.yml`).

### Rationale

- Consistent with how Azure support works (zero code in this action)
- Zero upstream merge conflict risk
- Zero maintenance burden
- Maximum flexibility (users control their own auth method: WIF, SA key, etc.)
- Clean separation of concerns (auth is not this action's job)
- The downstream plan/apply actions would also need GCP auth for the feature to be complete

### Trade-offs

- Users must add `google-github-actions/auth@v2` as a prior step in their workflow
- Slightly less convenient than built-in auth (one extra workflow step)

## Changes

### File: `README.yaml`

1. **Line 61**: Update support statement from "AWS and Azure" to "AWS, Azure, and GCP"
2. **After line 113**: Add `#### GCP` section containing:
   - `atmos.yaml` config block with GCS/Firestore artifact-storage keys (kebab-case)
   - Full workflow example showing `google-github-actions/auth@v2` pre-authentication
   - Note about `id-token: write` permission requirement

### Config keys (kebab-case, matching existing YAML convention)

```yaml
artifact-storage:
  plan-repository-type: gcs
  bucket: my-gcs-terraform-plans
  metadata-repository-type: firestore
  gcp-project-id: my-gcp-project
  gcp-firestore-database-name: "(default)"
  gcp-firestore-collection-name: terraform-plan-storage
```

### Files NOT modified

- `action.yml` - no code changes
- Test fixtures/workflows - no GCP test infrastructure
- No new inputs or outputs

## Agent Team Research Summary

- **AWS Agent**: Identified 2 AWS-specific config extractions + 1 auth step in action.yml
- **Azure Agent**: Confirmed Azure support is entirely passive (documentation-only, zero code)
- **Upstream Agent**: No GCP issues, PRs, or plans found in cloudposse ecosystem
- **GCP Agent**: Mapped `google-github-actions/auth@v2` inputs/outputs and WIF setup
- **Devil's Advocate**: Recommended documentation-only approach; flagged config schema, testing,
  and downstream chain risks that are all avoided by this approach
