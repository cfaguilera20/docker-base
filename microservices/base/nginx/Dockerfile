FROM nginx:stable

RUN apt-get update \
    && apt-get install -y \
        bc \
        curl \
        unzip \
    && rm -rf /var/lib/apt/lists/*

ENV CONSUL_TEMPLATE_VERSION 0.16.0
ENV CONSUL_TEMPLATE_SHA1 064b0b492bb7ca3663811d297436a4bbf3226de706d2b76adade7021cd22e156

RUN curl --retry 7 -Lso /tmp/consul-template.zip \
        "https://releases.hashicorp.com/consul-template/${CONSUL_TEMPLATE_VERSION}/consul-template_${CONSUL_TEMPLATE_VERSION}_linux_amd64.zip" \
    && echo "${CONSUL_TEMPLATE_SHA1}  /tmp/consul-template.zip" | sha256sum -c \
    && unzip /tmp/consul-template.zip -d /usr/local/bin \
    && rm /tmp/consul-template.zip

ENV CONTAINERPILOT_VERSION 2.4.3
ENV CONTAINERPILOT_SHA1 2c469a0e79a7ac801f1c032c2515dd0278134790
ENV CONTAINERPILOT file:///etc/containerpilot.json

RUN curl --retry 7 -Lso /tmp/containerpilot.tar.gz \
         "https://github.com/joyent/containerpilot/releases/download/${CONTAINERPILOT_VERSION}/containerpilot-${CONTAINERPILOT_VERSION}.tar.gz" \
    && echo "${CONTAINERPILOT_SHA1}  /tmp/containerpilot.tar.gz" | sha1sum -c \
    && tar zxf /tmp/containerpilot.tar.gz -C /usr/local/bin \
    && rm /tmp/containerpilot.tar.gz

COPY config/ /etc
COPY scripts/ /usr/local/bin

RUN chmod +x /usr/local/bin/reload.sh
RUN chmod +x /usr/local/bin/sensor.sh

RUN echo 01 > ca.srl \
    && openssl genrsa -out ca-key.pem 2048 \
    && openssl req -new -x509 -days 365 -subj "/CN=*" -key ca-key.pem -out ca.pem \
    && openssl genrsa -out server-key.pem 2048 \
    && openssl req -subj "/CN=*" -new -key server-key.pem -out server.csr \
    && openssl x509 -req -days 365 -in server.csr -CA ca.pem -CAkey ca-key.pem -out server-cert.pem \
    && openssl rsa -in server-key.pem -out server-key.pem \
    && cp *.pem /etc/nginx/ \
    && cp *.csr /etc/nginx/

CMD [ "/usr/local/bin/containerpilot", \
    "nginx", \
        "-g", \
        "daemon off;"]