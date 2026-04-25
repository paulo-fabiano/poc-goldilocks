#scratch
FROM alpine:3.22.4

RUN apk add curl jq

RUN curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

RUN chmod +x kubectl

RUN mv kubectl /usr/bin/kubectl

USER 10001