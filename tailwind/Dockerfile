FROM debian:bookworm-slim AS downloader
ARG TARGETARCH

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

# Deliberately the glibc asset, not the -musl one: every image that consumes
# this binary is Debian based, and the musl build will not run there.
RUN TWARCH=$([ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "x64") \
    && curl -fsSL "https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-${TWARCH}" -o /tailwindcss \
    && chmod +x /tailwindcss

FROM scratch
COPY --from=downloader /tailwindcss /usr/local/bin/tailwindcss
