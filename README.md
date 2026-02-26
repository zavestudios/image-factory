# Image Factory (Starter Kit)

This repository contains:
- `example-tenant-app/` : a minimal tenant repo that consumes shared workflows from `platform-pipelines`

Shared reusable workflows now live in the `platform-pipelines` repository.
Cluster policy source of truth now lives in the `gitops` repository under `platform/policies/kyverno/`.

## Operating Model

Build:
- One build produces an immutable image digest.
- The pipeline attaches SBOM + provenance to the image (BuildKit attestations).
- The pipeline signs the image with keyless Cosign (GitHub OIDC identity).
- Signing and verification should target image digests (`image@sha256:...`), while tags are treated as mutable pointers.

Promote:
- Promotion is tagging an existing digest with an environment tag (no rebuild).
- Environments should deploy only from environment tags (`dev`, `staging`, `prod`).

Enforce:
- Kyverno verifies signatures at admission time.
- Unsigned images are blocked once policy is set to `Enforce`.

## Roadmap (platform maturity)

Phase 0 — Prove end-to-end
- Build -> scan -> sign -> deploy with Kyverno in `Audit` mode.
- Verify signing/verification against image digests (`image@sha256:...`), not mutable tags.
- Switch policy to `Enforce` after validation.

Phase 0.5 — Platform prerequisites
- Enable GitHub OIDC for Actions and confirm `id-token: write` is allowed.
- Configure GHCR permissions for publish/promote workflows.
- Ensure tenant repos can call reusable workflows from the shared pipeline repo.
- Define GitHub Environments (`dev`, `staging`, `prod`) and protection rules (required reviewers for `staging`/`prod`).

Phase 1 — Determinism + CI integrity
- Pin base images by digest.
- Standardize build args, labels, and caching.
- Add Renovate/Dependabot to keep base digests current.
- Pin third-party GitHub Actions by commit SHA (not only version tags) in shared workflows.

Phase 2 — Promotion + environment governance
- Require digest-based promotion for all non-dev environments.
- Allow only `dev|staging|prod` environment tags in promotion workflows.
- Enforce GitHub Environment approval gates for `staging` and `prod`.
- Define promoter roles and retain auditable promotion history.

Phase 3 — Attestation enforcement
- Choose enforcement method via a Kyverno compatibility matrix:
  - Cosign attestation verification path, or
  - Sigstore bundle verification path.
- Validate cluster/version support before enforcing.
- Enforce required attestation presence/type (SBOM + provenance) once validated.

Phase 4 — Registry sovereignty (optional)
- Mirror signed images/artifacts to Harbor (or equivalent) for private/air-gapped posture.
- Preserve build/signing identity and artifact integrity across replication.


## Reusable workflow usage

Use the shared workflows from `platform-pipelines` in tenant repositories:

```yaml
jobs:
  build:
    uses: YOUR_ORG/platform-pipelines/.github/workflows/container-build.yml@v0
    with:
      image_name: YOUR_ORG/your-app
      context: .
    secrets:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

```yaml
jobs:
  promote:
    uses: YOUR_ORG/platform-pipelines/.github/workflows/container-promote.yml@v0
    with:
      image: ghcr.io/YOUR_ORG/your-app
      digest: sha256:...
      environment: staging
    secrets:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Version pinning strategy

Use release channels, not `@main`, to prevent unexpected breaking changes.

- Maintain tags in `platform-pipelines`:
  - `v0` (moving tag for latest compatible within major version 0)
  - `v0.1.0`, `v0.1.1`, ... (immutable semver tags)
- Tenant repos pin to:
  - `@v0` for controlled updates
  - or `@v0.x.y` for strict immutability

Platform team update process:
1) Cut a new semver tag (for example, `v0.2.0`).
2) Move `v0` to point at it after validation.
3) Optionally automate tenant bumps with Renovate/Dependabot.

## Explicit scope boundary

This starter kit provides reusable build/sign/promote workflows and a Kyverno verification policy example.
Application deployment manifests/controllers (Helm/Kustomize/Argo CD/Flux) are intentionally out of scope.


## Policy source of truth

Kyverno runtime policies are managed and applied via `gitops`, not this repository.
Current path: `gitops/platform/policies/kyverno/require-signed-ghcr-images.yaml`.

Rollout model:
- Start with `validationFailureAction: Audit`
- Validate signed/unsigned behavior
- Move to `Enforce` after validation

## Setup checklist

1) Confirm platform prerequisites:
- OIDC enabled and `id-token: write` available in workflows.
- GHCR permissions configured for publish/promotion.
- Reusable workflow access enabled for tenant repos.
- Environments and protection rules configured (`dev`, `staging`, `prod`).

2) In `platform-pipelines`, publish workflow tags:
- `v0.1.0` (immutable)
- `v0` (moving)

3) Update tenant repos to call `@v0`.

4) Apply Kyverno policy:
- Start in `Audit`, then move to `Enforce`.

5) Teach teams:
- Deploy by env tags
- Promote by digest
- Never rebuild for prod
