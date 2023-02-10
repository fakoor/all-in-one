FROM coturn/coturn:4.6.1-r1-alpine

COPY --from=nats:2.9.14-scratch /nats-server /usr/local/bin/nats-server
COPY --from=nats:2.9.14-scratch /nats-server.conf /etc/nats/nats.conf

COPY --from=strukturag/nextcloud-spreed-signaling:1.0.0 /usr/bin/nextcloud-spreed-signaling /usr/local/bin/signaling

USER root
RUN apk add --no-cache ca-certificates wget tzdata bash shadow \
                       openssl supervisor netcat-openbsd; \
    apk add --no-cache janus-gateway --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing; \
    useradd --system talk; \
    apk del --no-cache shadow;

# Give root a random password
RUN echo "root:$(openssl rand -base64 12)" | chpasswd

COPY --chmod=775 start.sh /usr/bin/
COPY --chmod=664 supervisord.conf /
RUN cp /etc/janus/janus.jcfg.sample /etc/janus/janus.jcfg; \
    cp /etc/janus/janus.transport.websockets.jcfg.sample /etc/janus/janus.transport.websockets.jcfg; \
    cp /etc/janus/janus.transport.mqtt.jcfg.sample /etc/janus/janus.transport.mqtt.jcfg; \
    touch /etc/turnserver.conf;

RUN wget https://raw.githubusercontent.com/rxi/json.lua/master/json.lua -O /usr/share/janus/lua/json.lua; \
    wget https://raw.githubusercontent.com/kikito/ansicolors.lua/master/ansicolors.lua -O /usr/share/janus/lua/ansicolors.lua

RUN mkdir -vp /etc/nats \
              /var/tmp \
              /var/lib/turn \
              /etc/signaling \
              /var/log/supervisord \
              /var/run/supervisord; \
    chown talk:talk -R /etc \
                       /usr \
                       /var/lib/turn \
                       /var/log/supervisord \
                       /var/run/supervisord; \
    echo "listen: 127.0.0.1:4222" > /etc/nats/nats.conf;

# Set default talk port https://github.com/nextcloud/all-in-one/issues/1011
ENV TALK_PORT=3478

USER talk
ENTRYPOINT ["start.sh"]
CMD ["/usr/bin/supervisord", "-c", "/supervisord.conf"]

HEALTHCHECK CMD (nc -z localhost 8081 && nc -z localhost 8188 && nc -z localhost 4222 && nc -z localhost $TALK_PORT) || exit 1
LABEL com.centurylinklabs.watchtower.monitor-only="true"
