# Example Tenant App

This is a minimal workload repo that consumes the shared container pipeline.

What to do:
1) Replace `YOUR_ORG` in `.github/workflows/*.yml`.
2) Push to `main` to build and publish `ghcr.io/YOUR_ORG/example-tenant-app:sha-<short>`.
3) Use the Promote workflow (workflow_dispatch) to tag a digest as `dev|staging|prod`.
4) Protect `staging` and `prod` with GitHub Environment approvals before promotion.

Deployment guidance:
- Deploy using `:dev`, `:staging`, or `:prod` tags.
- Promotion moves those tags to point at an existing digest; it does not rebuild.
- Use digest (`image@sha256:...`) as the immutable source of truth for verification and audit.
