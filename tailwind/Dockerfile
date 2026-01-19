FROM alpine:latest AS downloader
ARG TARGETARCH

RUN TWARCH=$([ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "x64") \
    && wget -q "https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-${TWARCH}" -O /tailwindcss \
    && chmod +x /tailwindcss

FROM scratch
COPY --from=downloader /tailwindcss /usr/local/bin/tailwindcss
