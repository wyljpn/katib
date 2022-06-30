# 20220630 Yulong
# katib-controller
# Golang in golang:alpine is 1.18.3

# Build the Katib controller.
#FROM golang:alpine AS build-env
FROM alpine:latest AS build-env

# Install Go
ARG GOLANG_VERSION=1.18.3

RUN apk update && apk add go gcc bash musl-dev openssl-dev ca-certificates && update-ca-certificates

RUN wget https://dl.google.com/go/go$GOLANG_VERSION.src.tar.gz && tar -C /usr/local -xzf go$GOLANG_VERSION.src.tar.gz

RUN cd /usr/local/go/src && ./make.bash

ENV PATH=$PATH:/usr/local/go/bin

RUN rm go$GOLANG_VERSION.src.tar.gz

#we delete the apk installed version to avoid conflict
RUN apk del go

RUN go version

WORKDIR /go/src/github.com/kubeflow/katib

# Download packages.
COPY go.mod .
COPY go.sum .
RUN go mod download -x

# Copy sources.
COPY cmd/ cmd/
COPY pkg/ pkg/

# Build the binary.
RUN if [ "$(uname -m)" = "ppc64le" ]; then \
    CGO_ENABLED=0 GOOS=linux GOARCH=ppc64le go build -a -o katib-controller  ./cmd/katib-controller/v1beta1; \
    elif [ "$(uname -m)" = "aarch64" ]; then \
    CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -a -o katib-controller  ./cmd/katib-controller/v1beta1; \
    else \
    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -o katib-controller  ./cmd/katib-controller/v1beta1; \
    fi

# Copy the controller-manager into a thin image.
FROM alpine:latest
WORKDIR /app
COPY --from=build-env /go/src/github.com/kubeflow/katib/katib-controller .
ENTRYPOINT ["./katib-controller"]
