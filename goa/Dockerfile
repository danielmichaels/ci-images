FROM golang:1.24-bookworm

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    protobuf-compiler
RUN go install goa.design/goa/v3/cmd/goa@v3 && \
    go install  github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest && \
    go install	github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger@latest && \
    go install	github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest && \
    go install	github.com/golang/protobuf/protoc-gen-go@latest

