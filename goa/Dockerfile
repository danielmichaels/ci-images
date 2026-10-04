FROM debian:bookworm-slim AS protoc
ARG TARGETARCH

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    ca-certificates curl unzip \
    && rm -rf /var/lib/apt/lists/*

# Debian's protobuf-compiler is stuck on 3.21.12 and ships only the binary, so
# importing google/protobuf/timestamp.proto fails. The upstream zip carries the
# well-known types alongside it. The version appears in the asset name, so the
# tag is resolved from the releases/latest redirect rather than the API, which
# is rate limited per IP and CI runners share addresses.
RUN set -eu; \
    ARCH=$([ "$TARGETARCH" = "arm64" ] && echo aarch_64 || echo x86_64); \
    VER=$(curl -fsSLI -o /dev/null -w '%{url_effective}' https://github.com/protocolbuffers/protobuf/releases/latest | sed 's#.*/tag/v##'); \
    curl -fsSL "https://github.com/protocolbuffers/protobuf/releases/download/v${VER}/protoc-${VER}-linux-${ARCH}.zip" -o /tmp/protoc.zip; \
    unzip -q /tmp/protoc.zip 'bin/*' 'include/*' -d /out

FROM golang:1.27-bookworm

# goa publishes no prebuilt binaries, so these are compiled. Keeping the RUN
# ahead of the COPY lets buildkit fetch protoc concurrently with the compile
# rather than after it.
# protoc-gen-swagger is grpc-gateway v1 and superseded by protoc-gen-openapiv2;
# kept only so existing generate targets that invoke it by name keep working.
RUN go install goa.design/goa/v3/cmd/goa@v3 && \
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest && \
    go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest && \
    go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest && \
    go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger@latest

COPY --from=protoc /out/bin/protoc /usr/bin/protoc
COPY --from=protoc /out/include/google /usr/include/google
