FROM danielmichaels/ci-goose as goose
FROM danielmichaels/ci-taskfile as taskfile
FROM golang:1.22-bookworm

COPY --from=goose ["/usr/local/bin/goose", "/usr/local/bin/goose"]
COPY --from=taskfile ["/usr/local/bin/task", "/usr/local/bin/task"]
