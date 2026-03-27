# Prebuild Keycloak so runtime startup can use `start --optimized`
# without triggering an in-cluster Quarkus rebuild.
FROM quay.io/keycloak/keycloak:26.5.5 AS builder

ENV KC_DB=postgres
ENV KC_HEALTH_ENABLED=true
ENV KC_METRICS_ENABLED=true
ENV KC_HTTP_RELATIVE_PATH=/auth
ENV KC_HTTPS_CLIENT_AUTH=request

RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:26.5.5

COPY --from=builder /opt/keycloak/ /opt/keycloak/
