# Build websocketd for the target architecture
FROM golang:alpine AS wsbuild
ARG WEBSOCKETD_VERSION=v0.4.1
RUN apk add --no-cache git \
 && git clone --depth 1 --branch ${WEBSOCKETD_VERSION} https://github.com/joewalnes/websocketd.git /src
WORKDIR /src
ARG GORILLA_WEBSOCKET_VERSION=v1.5.3
RUN set -eu; \
    if [ ! -f go.mod ]; then go mod init github.com/joewalnes/websocketd; fi; \
    go get github.com/gorilla/websocket@${GORILLA_WEBSOCKET_VERSION}; \
    go mod tidy; \
    CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /websocketd .; \
    go version -m /websocketd

# Final image
FROM alpine:3
ADD ./zoo-project/zoo-services/utils/open-api/server/subscriber.py /shell.py
RUN apk update && apk add --no-cache python3 expat zlib py3-pip py3-setuptools py3-wheel py3-redis \
    && apk add --upgrade libcrypto3 libssl3 \
    && chown nobody: /shell.py \
    && if [ ! -e /usr/bin/python ]; then ln -sf python3 /usr/bin/python ; fi \
    && if [ ! -e /usr/bin/pip ]; then ln -s pip3 /usr/bin/pip ; fi \
    && apk del py3-pip py3-setuptools
COPY --from=wsbuild /websocketd /websocketd/websocketd
WORKDIR /websocketd
ENTRYPOINT [ "/websocketd/websocketd" ]
CMD [ "--port=8888", "/shell.py" ]
