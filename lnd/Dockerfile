FROM golang:1.13-alpine as builder

WORKDIR /go/src/github.com/lightningnetwork/lnd

# Force Go to use the cgo based DNS resolver. This is required to ensure DNS
# queries required to connect to linked containers succeed.
ENV GODEBUG netdns=cgo

RUN apk add --no-cache --update alpine-sdk git make \
  && git clone -n https://github.com/lightningnetwork/lnd . \
  && git checkout v0.8.2-beta \
  && make \
  && make install tags="autopilotrpc chainrpc invoicesrpc routerrpc signrpc walletrpc watchtowerrpc wtclientrpc"

# Start a new, final image to reduce size.
FROM alpine as final

RUN adduser -S lnd
RUN apk add --no-cache \
  bash \
  su-exec

ENV LND_DATA=/home/lnd/.lnd
COPY --from=builder /go/bin/lncli /bin/
COPY --from=builder /go/bin/lnd /bin/
COPY docker-entrypoint.sh /entrypoint.sh

VOLUME ["/home/lnd/.lnd"]

ADD ./lnd.conf /home/lnd/.lnd/lnd.conf

ENTRYPOINT ["/entrypoint.sh"]

CMD ["lnd"]