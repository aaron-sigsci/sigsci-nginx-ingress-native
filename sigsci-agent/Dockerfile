FROM alpine:latest

RUN apk add --no-cache curl
RUN curl https://dl.signalsciences.net/sigsci-agent/3.21.0/linux/sigsci-agent_3.21.0.tar.gz | tar -zxf - -C /usr/sbin/

ENTRYPOINT ["/usr/sbin/sigsci-agent"]
