FROM danielmichaels/ci-goose AS goose
FROM danielmichaels/ci-taskfile AS taskfile
FROM danielmichaels/ci-goa AS goa
FROM danielmichaels/ci-templ AS templ
FROM danielmichaels/ci-linters AS linters
FROM golangci/golangci-lint:latest AS golangci

FROM golang:1.24-bookworm

COPY --from=goose ["/usr/local/bin/goose", "/usr/local/bin/goose"]
COPY --from=goa ["/go/bin/goa", "/usr/local/bin/goa"]
COPY --from=templ ["/go/bin/templ", "/usr/local/bin/templ"]
COPY --from=taskfile ["/usr/local/bin/task", "/usr/local/bin/task"]
COPY --from=linters ["/go/bin/betteralign", "/usr/local/bin/betteralign"]
COPY --from=linters ["/go/bin/gofumpt", "/usr/local/bin/gofumpt"]
COPY --from=linters ["/go/bin/golines", "/usr/local/bin/golines"]
COPY --from=golangci ["/usr/bin/golangci-lint", "/usr/local/bin/golangci-lint"]
