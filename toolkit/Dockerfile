FROM danielmichaels/ci-goose as goose
FROM danielmichaels/ci-taskfile as taskfile
FROM danielmichaels/ci-goa as goa
FROM danielmichaels/ci-templ as templ

FROM golang:1.22-bookworm

COPY --from=goose ["/usr/local/bin/goose", "/usr/local/bin/goose"]
COPY --from=goa ["/go/bin/goa", "/usr/local/bin/goa"]
COPY --from=templ ["/go/bin/templ", "/usr/local/bin/templ"]
COPY --from=taskfile ["/usr/local/bin/task", "/usr/local/bin/task"]
