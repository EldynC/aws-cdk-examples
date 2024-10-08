FROM --platform=linux/amd64 public.ecr.aws/amazonlinux/amazonlinux:2 AS builder
RUN yum update -y && \
    yum install -y ca-certificates unzip tar gzip git && \
    yum clean all && \
    rm -rf /var/cache/yum

RUN curl -LO https://golang.org/dl/go1.17.1.linux-amd64.tar.gz && \
    tar -C /usr/local -xzvf go1.17.1.linux-amd64.tar.gz

ENV PATH="${PATH}:/usr/local/go/bin"
ENV GOPATH="${HOME}/go"
ENV PATH="${PATH}:${GOPATH}/bin"

# ARG GO_PROXY=https://proxy.golang.org
ARG GO_PROXY=direct

WORKDIR /go/src/github.com/aws/aws-app-mesh-examples/colorapp/gateway

# go.mod and go.sum go into their own layers.
COPY go.mod .
COPY go.sum .

# Set the proxies for the go compiler
# RUN go env -w GOPROXY=${GO_PROXY} -- This line is commented out in favor of GOPRIVATE, but if GOPROXY works on your machine you can use this line instead.
RUN go env -w GOPRIVATE=*
# This ensures `go mod download` happens only when go.mod and go.sum change.
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix nocgo -o /aws-app-mesh-examples-colorapp-gateway .

# Because this was built on an M1 Mac, you may be able to remove the --platform flag
FROM --platform=linux/amd64 public.ecr.aws/amazonlinux/amazonlinux:2
RUN yum update -y && \
    yum install -y ca-certificates && \
    yum clean all && \
    rm -rf /var/cache/yum

COPY --from=builder /aws-app-mesh-examples-colorapp-gateway /bin/aws-app-mesh-examples-colorapp-gateway

ENTRYPOINT ["/bin/aws-app-mesh-examples-colorapp-gateway"]
