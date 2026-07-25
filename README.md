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
