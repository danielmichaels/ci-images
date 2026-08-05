FROM debian:bookworm-slim AS downloader
ARG TARGETARCH

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

# Task depends on hashicorp/go-getter for remote taskfiles, which pulls the AWS
# and GCP SDKs into the build: ~127 modules. Compiling that for the emulated
# arm64 leg took over 20 minutes, so take the release binary instead.
RUN mkdir -p /out \
    && curl -fsSL "https://github.com/go-task/task/releases/latest/download/task_linux_${TARGETARCH}.tar.gz" \
    | tar -xz -C /out task

FROM golang:1.26-bookworm
COPY --from=downloader /out/task /go/bin/task
