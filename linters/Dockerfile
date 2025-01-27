FROM golang:1.23-bookworm

# /go/bin/betteralign
RUN go install github.com/dkorunic/betteralign/cmd/betteralign@latest
# /go/bin/golines
RUN go install github.com/segmentio/golines@latest
# /go/bin/gofumpt
RUN go install mvdan.cc/gofumpt@latest
