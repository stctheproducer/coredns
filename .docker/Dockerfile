ARG DEBIAN_IMAGE=debian:stable-slim
ARG BASE=gcr.io/distroless/static-debian11:nonroot

FROM golang:1.20-alpine AS base
WORKDIR /app

RUN apk --no-cache --no-progress add bash git openssh

COPY go.mod go.sum ./

RUN go mod download
COPY . .

RUN make

FROM --platform=$BUILDPLATFORM ${DEBIAN_IMAGE} AS build
SHELL [ "/bin/sh", "-ec" ]

RUN export DEBCONF_NONINTERACTIVE_SEEN=true \
           DEBIAN_FRONTEND=noninteractive \
           DEBIAN_PRIORITY=critical \
           TERM=linux ; \
    apt-get -qq update ; \
    apt-get -yyqq upgrade ; \
    apt-get -yyqq install ca-certificates libcap2-bin; \
    apt-get clean
COPY --from=base /app/coredns /coredns
RUN setcap cap_net_bind_service=+ep /coredns

FROM --platform=$TARGETPLATFORM ${BASE}
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build /coredns /coredns
USER nonroot:nonroot
EXPOSE 53 53/udp
ENTRYPOINT ["/coredns"]