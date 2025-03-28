FROM alpine:latest AS builder

ARG TARGETPLATFORM

COPY . /tmp/artifacts
RUN mkdir -p /tmp/output; \
    cd /tmp/artifacts; \
    ARTIFACT_ARCH=""; \
    if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
        ARTIFACT_ARCH="x86_64"; \
    elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then \
        ARTIFACT_ARCH="aarch64"; \
    else \
        echo "Unsupported architecture: $TARGETARCH"; \
        exit 1; \
    fi; \
    cp /tmp/artifacts/easytier-linux-${ARTIFACT_ARCH}/* /tmp/output

FROM alpine:latest

RUN apk add --no-cache tzdata runit

ENV TZ=Asia/Shanghai
WORKDIR /app

COPY --from=builder --chmod=755 /tmp/output/easytier-core /usr/local/bin/
COPY --from=builder --chmod=755 /tmp/output/easytier-cli /usr/local/bin/
COPY --from=builder --chmod=755 /tmp/output/easytier-web /usr/local/bin/

RUN mkdir -p /etc/service/easytier-core /etc/service/easytier-web

# core service run script
RUN printf '#!/bin/sh\nexec /usr/local/bin/easytier-core $CORE_ARGS\n' > /etc/service/easytier-core/run && \
    chmod +x /etc/service/easytier-core/run

# web service run script with graceful disable
RUN printf '#!/bin/sh\n\
if [ "$EASYTIER_WEB_ENABLED" = "true" ]; then\n\
  exec /usr/local/bin/easytier-web $WEB_ARGS\n\
else\n\
  echo "easytier-web is disabled"\n\
  exec tail -f /dev/null\n\
fi\n' > /etc/service/easytier-web/run && \
    chmod +x /etc/service/easytier-web/run

EXPOSE 11010/tcp 11010/udp 11011/udp 11011/tcp 11012/tcp

ENTRYPOINT ["runsvdir", "-P", "/etc/service"]
