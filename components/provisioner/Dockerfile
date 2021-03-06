FROM golang:1.13.1-alpine3.10 as builder

ENV BASE_APP_DIR /go/src/github.com/kyma-incubator/compass/components/provisioner
WORKDIR ${BASE_APP_DIR}

#
# Copy files
#

ENV GO111MODULES=on

COPY ./assets/ /assets/
COPY ./internal/ ${BASE_APP_DIR}/internal/
COPY ./pkg/ ${BASE_APP_DIR}/pkg/
COPY ./cmd/ ${BASE_APP_DIR}/cmd
COPY ./licenses ${BASE_APP_DIR}/licenses
COPY ./go.mod ${BASE_APP_DIR}/go.mod
COPY ./go.sum ${BASE_APP_DIR}/go.sum

RUN apk add git && go mod download

#
# Add ca-certificates
#

RUN apk add -U --no-cache ca-certificates && update-ca-certificates

#
# Build app
#

RUN go build -v -o main ./cmd/
RUN mkdir /app && mv ./main /app/main && mv ./licenses /app/licenses
RUN wget https://github.com/kyma-incubator/terraform-provider-gardener/releases/download/v0.0.2/terraform-provider-gardener-linux-amd64

FROM alpine:3.10
LABEL source = git@github.com:kyma-incubator/compass.git
# Hydroform needs write permissions
RUN mkdir /app && chmod 777 /app
WORKDIR /app

#
# Copy binary
#

COPY --from=builder /assets /assets
COPY --from=builder /app /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=builder /go/src/github.com/kyma-incubator/compass/components/provisioner/terraform-provider-gardener-linux-amd64 /.terraform.d/plugins/linux_amd64/terraform-provider-gardener

RUN chmod +x /.terraform.d/plugins/linux_amd64/terraform-provider-gardener

#
# Run app
#

CMD ["/app/main"]