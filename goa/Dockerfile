FROM golang:1.26-bookworm

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install --no-install-recommends --assume-yes \
    protobuf-compiler
# protoc-gen-swagger is grpc-gateway v1 and superseded by protoc-gen-openapiv2;
# kept only so existing generate targets that invoke it by name keep working.
RUN go install goa.design/goa/v3/cmd/goa@v3 && \
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest && \
    go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-grpc-gateway@latest && \
    go install github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2@latest && \
    go install github.com/grpc-ecosystem/grpc-gateway/protoc-gen-swagger@latest
