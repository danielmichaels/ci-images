FROM debian:bookworm-slim AS downloader
ARG TARGETARCH

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

# templ names its amd64 asset x86_64.
RUN ARCH=$([ "$TARGETARCH" = "arm64" ] && echo arm64 || echo x86_64) \
    && mkdir -p /out \
    && curl -fsSL "https://github.com/a-h/templ/releases/latest/download/templ_Linux_${ARCH}.tar.gz" \
    | tar -xz -C /out templ

FROM golang:1.27-bookworm
COPY --from=downloader /out/templ /go/bin/templ
