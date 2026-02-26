# Example minimal app image.
# In a hardened phase, pin base by digest (e.g., alpine@sha256:...).
FROM alpine:3.19

RUN adduser -D appuser
USER appuser

CMD ["sh", "-c", "echo Hello from a signed image; sleep 3600"]
