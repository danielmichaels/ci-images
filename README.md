# ci-images

Prebuilt container images carrying the Go toolchain and code-generation tools I
use across projects, so CI jobs don't spend minutes on `go install` every run.

Each image is a thin layer over `golang:1.26-bookworm` containing one tool (or
one family of tools). `ci-toolkit` gathers all of them into a single image.

## Registries

Images are published to **GitHub Container Registry** and mirrored to Docker Hub.

| Registry | Prefix | Status |
| --- | --- | --- |
| GHCR | `ghcr.io/danielmichaels/` | Preferred — use this for new work |
| Docker Hub | `danielmichaels/` | Maintained for backwards compatibility |

Both registries receive identical content and tags from the same build. Docker
Hub is a mirror, not a fallback — if you are writing something new, point it at
GHCR.

### Tags

| Tag | Meaning |
| --- | --- |
| `latest` | Most recent weekly build. Tools move underneath you. |
| `YYYY-MM-DD` | Immutable snapshot of one weekly build. |

Every tool is installed at `@latest`, and the images rebuild from scratch every
Sunday, so `:latest` deliberately drifts. **Pin a date tag anywhere a
reproducible build matters** — a date-tagged `ci-toolkit` is always assembled
from the base images carrying the same date tag.

```dockerfile
FROM ghcr.io/danielmichaels/ci-toolkit:2026-08-05    # reproducible
FROM ghcr.io/danielmichaels/ci-toolkit:latest        # always current
```

## Images

All images are `linux/amd64` and `linux/arm64`.

| Image | Binaries | Path |
| --- | --- | --- |
| `ci-goose` | `goose` | `/usr/local/bin/goose` |
| `ci-taskfile` | `task` | `/go/bin/task` |
| `ci-templ` | `templ` | `/go/bin/templ` |
| `ci-linters` | `betteralign`, `gofumpt`, `golines` | `/go/bin/` |
| `ci-tailwind` | `tailwindcss` | `/usr/local/bin/tailwindcss` |
| `ci-goa` | `goa`, `protoc` | `/go/bin/goa`, `/usr/bin/protoc` |
| | `protoc-gen-go`, `protoc-gen-grpc-gateway`, `protoc-gen-openapiv2`, `protoc-gen-swagger` | `/go/bin/` |
| `ci-toolkit` | everything below | `/usr/local/bin/` |

`ci-toolkit` normalises every binary into `/usr/local/bin`, so you get one
predictable path regardless of which image a tool originally came from:

```
/usr/local/bin/goose         /usr/local/bin/gofumpt
/usr/local/bin/goa           /usr/local/bin/golines
/usr/local/bin/templ         /usr/local/bin/golangci-lint
/usr/local/bin/task          /usr/local/bin/tailwindcss
/usr/local/bin/betteralign
```

Two things worth knowing:

- `golangci-lint` is **only** in `ci-toolkit` (pulled from the upstream
  `golangci/golangci-lint` image); there is no `ci-golangci`.
- `ci-toolkit` takes `goa` but **not** `protoc` or the `protoc-gen-*` plugins.
  If you generate protobuf code, use `ci-goa` or copy those binaries across
  explicitly.

`ci-tailwind` is built `FROM scratch`. It holds a single static binary and has
no shell, so it exists purely as a `COPY --from` source — `docker run` on it
will not work.

## Using the images

### As a CI job container

The whole image becomes your build environment. Everything is already on `PATH`:

```yaml
# GitHub Actions
jobs:
  generate:
    runs-on: ubuntu-latest
    container: ghcr.io/danielmichaels/ci-toolkit:latest
    steps:
      - uses: actions/checkout@v6
      - run: task generate
      - run: golangci-lint run
```

```yaml
# Woodpecker CI
steps:
  generate:
    image: ghcr.io/danielmichaels/ci-toolkit:latest
    commands:
      - templ generate
      - task test
```

Or interactively:

```sh
docker run --rm -v "$PWD":/src -w /src \
  ghcr.io/danielmichaels/ci-toolkit:latest task build
```

### Copying binaries into your own image

This is the main reason these images exist. Rather than inheriting a large
toolkit image, name it as a build stage and copy out only the binaries you
need — the stage is discarded and contributes nothing to your final image size.

```dockerfile
# syntax=docker/dockerfile:1
FROM ghcr.io/danielmichaels/ci-toolkit:2026-08-05 AS tools

FROM golang:1.26-bookworm AS build
COPY --from=tools /usr/local/bin/templ /usr/local/bin/templ
COPY --from=tools /usr/local/bin/task  /usr/local/bin/task

WORKDIR /src
COPY . .
RUN templ generate && task build

FROM gcr.io/distroless/base-debian12
COPY --from=build /src/bin/app /app
ENTRYPOINT ["/app"]
```

Only `templ` and `task` are pulled in, only during the build stage, and the
shipped image contains neither.

Pull from a single-tool image when you need exactly one thing — it is a smaller
download than the full toolkit:

```dockerfile
FROM ghcr.io/danielmichaels/ci-tailwind:latest AS tailwind

FROM node:22-bookworm AS assets
COPY --from=tailwind /usr/local/bin/tailwindcss /usr/local/bin/tailwindcss
RUN tailwindcss -i input.css -o output.css --minify
```

Note the source paths differ between the single-tool images and `ci-toolkit`
(`/go/bin/task` vs `/usr/local/bin/task`) — see the table above.

#### Requirements for the target image

**Assume the target image needs glibc.** Everything is built on Debian
bookworm, but the linkage is mixed rather than uniform:

| Statically linked | Dynamically linked (needs glibc) |
| --- | --- |
| `goose`, `gofumpt`, `betteralign`, `golangci-lint` | `goa`, `templ`, `task`, `golines`, `tailwindcss` |

`go install` only links against libc when a package pulls in something like
`net` or `os/user`, which is why the split looks arbitrary. It can flip when a
tool picks up a new dependency, so treat the table as a snapshot and design for
the glibc case.

So the target must be glibc-based: Debian, Ubuntu, or distroless `*-debian12`.
On Alpine or any other musl image, a dynamically linked binary fails with
`not found` — reported against the binary itself even though the file is
plainly there, because what is actually missing is the ELF loader.

If your target must be Alpine, install `gcompat`, or build the tool inside your
own image rather than copying it.

Beyond libc there is nothing to satisfy: no config files, no runtime data
directory, no environment variables.

#### Architectures

Both `linux/amd64` and `linux/arm64` are published, and Docker resolves the
matching variant automatically. In a multi-platform build (`docker buildx build
--platform linux/amd64,linux/arm64`), a `COPY --from` stage defaults to the
**target** platform, which is what you want for binaries being shipped in the
final image.

If instead you need a tool to *execute* during the build, pin the stage to the
build host so it runs natively rather than under emulation:

```dockerfile
FROM --platform=$BUILDPLATFORM ghcr.io/danielmichaels/ci-toolkit:latest AS tools
```

## Development

`scripts/verify` builds every image for your host architecture and pushes
nothing. A failing image does not abort the run, so one invocation reports
everything that is broken:

```sh
./scripts/verify
```

```
building goose     ... ok
building taskfile  ... ok
building goa       ... FAILED  (log: /tmp/tmp.AbC123/goa.log)
building templ     ... ok
building linters   ... ok
building tailwind  ... ok
building toolkit   ... skipped (goa failed)

1 of 7 failed: goa
```

It exits non-zero if anything failed. Because `toolkit` copies from the other
six, it is skipped unless they all succeed.

### Publishing

Publishing is CI's job — there is no push script. `.github/workflows/docker-parallel.yml`
runs every Sunday at 00:00 UTC and builds the six base images in parallel
(`fail-fast: false`, so one broken tool does not hide the other five), then
builds `ci-toolkit` from them.

Trigger it by hand with:

```sh
gh workflow run docker-parallel.yml
gh run watch
```

### Adding a tool

1. Create `<tool>/Dockerfile` based on `golang:1.26-bookworm`. Do not use
   Alpine — the glibc guarantee above is what makes `COPY --from` work for
   consumers.
2. Add the directory name to the `image` matrix in the workflow and to
   `images` in `scripts/verify`.
3. If it belongs in the combined image, add a `COPY --from` line to
   `toolkit/Dockerfile` targeting `/usr/local/bin/`.
4. Run `./scripts/verify`.

`toolkit/Dockerfile` takes `REGISTRY` and `TAG` build args so it can be
assembled from somewhere other than GHCR `latest`:

```sh
docker build --build-arg REGISTRY=danielmichaels --build-arg TAG=latest ./toolkit
```
