# .github

Org-wide defaults and shared GitHub Actions workflows for `tiducto`.

## `generate-contract.yml` (reusable)

Shared contract-sync logic used by `spider-sdk-kotlin` and `spider-sdk-typescript`. Each SDK repo keeps its own thin caller workflow with its own triggers and calls this one:

```yaml
jobs:
  generate:
    uses: tiducto/.github/.github/workflows/generate-contract.yml@main
    with:
      add-paths: |
        <path-to-generated-output>
        contract/contract-version
    secrets: inherit
```

Caller contract: a `scripts/generate-contract.sh` script (accepting `--ref <sha>`), a `contract/contract-version` marker file, and a `CONTRACT_REPO_TOKEN` secret with read access to `tiducto/spider-contract` and `tiducto/spider-codegen`.

## `actions/gcp-auth` (composite)

Keyless GCP auth via Workload Identity Federation, shared by every repo that pushes/pulls Artifact Registry images or calls GCP APIs (`spider-services`, `spider-router`, `spider-bench`). The org forbids service-account keys, so CI impersonates a service account instead. Wire it with the org-level Actions variables (`WIF_PROVIDER`, `GCP_CI_SA`, `AR_HOST`) so no repo hardcodes the project, host, or provider:

```yaml
jobs:
  publish:
    permissions:
      contents: read
      id-token: write        # required for WIF
    steps:
      - uses: tiducto/.github/actions/gcp-auth@main
        with:
          workload_identity_provider: ${{ vars.WIF_PROVIDER }}
          service_account:            ${{ vars.GCP_CI_SA }}
          registry:                   ${{ vars.AR_HOST }}   # omit if the job only calls gcloud
          # setup_gcloud: 'true'                            # add for gcloud/gsutil/pulumi jobs
```

The GCP side (WIF pool/provider, org-scoped condition, per-repo impersonation bindings) is provisioned by `setup-github-wif-gcp.sh` in `spider-services`; the org variables by `setup-org-ci-vars.sh` there.
