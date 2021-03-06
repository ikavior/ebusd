
FROM debian:stretch as build

RUN apt-get update && apt-get install -y \
    libmosquitto-dev libstdc++6 libc6 libgcc1 \
    curl \
    autoconf automake g++ make git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build

ENV EBUSD_VERSION 21.1
ENV EBUSD_ARCH amd64

RUN git clone https://github.com/john30/ebusd.git /build \
    && ./make_all.sh

ARG UPLOAD_URL
ARG UPLOAD_CREDENTIALS
ARG UPLOAD_ONLY
RUN if [ -n "$UPLOAD_URL" ] && [ -n "$UPLOAD_CREDENTIALS" ]; then for img in ebusd-*.deb; do echo "upload $img"; curl -u "$UPLOAD_CREDENTIALS" -s -X POST --data-binary "@$img" -H "Content-Type: application/octet-stream" "$UPLOAD_URL/$img?a=$EBUSD_ARCH&v=$EBUSD_VERSION"; done; if [ -n "$UPLOAD_ONLY" ]; then echo "stopping for upload only"; exit 139; fi fi


FROM debian:stretch-slim

RUN apt-get update && apt-get install -y \
    logrotate libmosquitto1 libstdc++6 libc6 libgcc1 \
    && rm -rf /var/lib/apt/lists/*

LABEL maintainer "ebusd@ebusd.eu"

ENV EBUSD_VERSION 21.1
ENV EBUSD_ARCH amd64

LABEL version "${EBUSD_VERSION}-${EBUSD_ARCH}"

COPY --from=build /build/ebusd-*_mqtt1.deb ebusd.deb

RUN dpkg -i ebusd.deb \
    && rm -f ebusd.deb

EXPOSE 8888

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["-f", "--scanconfig"]
