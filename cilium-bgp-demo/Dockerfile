# syntax = docker/dockerfile:1.3
FROM golang:1.19-bullseye as builder
ARG GOBGP_VERSION=3.7.0
RUN curl -LO https://github.com/osrg/gobgp/releases/download/v${GOBGP_VERSION}/gobgp_${GOBGP_VERSION}_linux_amd64.tar.gz
RUN tar xzvf gobgp_${GOBGP_VERSION}_linux_amd64.tar.gz

FROM gcr.io/distroless/static:nonroot
COPY --from=builder /go/gobgpd /bin
COPY --from=builder /go/gobgp /bin
USER nonroot:nonroot
CMD ["/bin/gobgpd", "-f", "/bgp-conf.yaml", "-t", "yaml", "-l", "debug"]
