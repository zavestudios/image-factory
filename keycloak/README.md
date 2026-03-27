# Keycloak Image

This image prebuilds Keycloak for the ZaveStudios Big Bang deployment so the
runtime container can start with `start --optimized` and avoid in-cluster
Quarkus rebuilds.

## Published Image

- `ghcr.io/zavestudios/keycloak`

## Build-Time Configuration

The Dockerfile bakes in build-time options required by the deployment:

- `KC_DB=postgres`
- `KC_HEALTH_ENABLED=true`
- `KC_METRICS_ENABLED=true`

These are built into the image via:

```bash
/opt/keycloak/bin/kc.sh build
```

## Delivery Flow

1. Merge to `main` in `image-factory`
2. GitHub Actions builds and publishes `ghcr.io/zavestudios/keycloak:sha-<short>`
3. Promote the digest to `ghcr.io/zavestudios/keycloak:bootstrap`
4. `gitops` consumes the prebuilt image with `start --optimized`

## Notes

- This image is a platform image, not a tenant workload.
- Runtime config such as hostname and external DB connection stays in `gitops`.

