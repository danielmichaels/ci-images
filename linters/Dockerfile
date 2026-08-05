FROM debian:bookworm-slim AS downloader
ARG TARGETARCH

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

# gofumpt and golines put the version in the asset name, so the tag has to be
# resolved first. The releases/latest redirect is used rather than the API
# because the API is rate limited per IP and CI runners share addresses.
# betteralign names its amd64 asset x86_64; golines nests under a versioned
# directory; gofumpt ships a bare binary rather than a tarball.
RUN set -eu; \
    mkdir -p /out; \
    ALTARCH=$([ "$TARGETARCH" = "arm64" ] && echo arm64 || echo x86_64); \
    curl -fsSL "https://github.com/dkorunic/betteralign/releases/latest/download/betteralign_Linux_${ALTARCH}.tar.gz" \
      | tar -xz -C /out betteralign; \
    GOLINES=$(curl -fsSLI -o /dev/null -w '%{url_effective}' https://github.com/segmentio/golines/releases/latest | sed 's#.*/tag/##'); \
    curl -fsSL "https://github.com/segmentio/golines/releases/download/${GOLINES}/golines_${GOLINES#v}_linux_${TARGETARCH}.tar.gz" \
      | tar -xz -C /out --strip-components=1 --wildcards '*/golines'; \
    GOFUMPT=$(curl -fsSLI -o /dev/null -w '%{url_effective}' https://github.com/mvdan/gofumpt/releases/latest | sed 's#.*/tag/##'); \
    curl -fsSL "https://github.com/mvdan/gofumpt/releases/download/${GOFUMPT}/gofumpt_${GOFUMPT}_linux_${TARGETARCH}" -o /out/gofumpt; \
    chmod +x /out/gofumpt

FROM golang:1.26-bookworm
COPY --from=downloader /out/betteralign /go/bin/betteralign
COPY --from=downloader /out/golines /go/bin/golines
COPY --from=downloader /out/gofumpt /go/bin/gofumpt
