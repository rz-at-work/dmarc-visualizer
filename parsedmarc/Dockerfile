FROM alpine:3.20 as pkg-builder

RUN apk upgrade --no-cache && \
    apk -U add \
    sudo \
    alpine-sdk \
    apkbuild-cpan \
    perl-doc \
    perl-email-address-xs

RUN mkdir -p /var/cache/distfiles && \
    adduser -D packager && \
    addgroup packager abuild && \
    chgrp abuild /var/cache/distfiles && \
    chmod g+w /var/cache/distfiles && \
    echo "packager ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

WORKDIR /work
RUN chown packager /work
USER packager

RUN abuild-keygen -a -i -n

COPY --chown=packager:packager packages/ ./

RUN cd perl-io-all && \
    abuild -r && \
    cd ../perl-pod-usage && \
    abuild -r && \
    cd ../perl-throwable && \
    abuild -r && \
    cd ../perl-email-abstract && \
    abuild -r && \
    cd ../perl-email-sender && \
    abuild -r && \
    cd ../perl-email-outlook-message && \
    abuild -r

FROM python:3.11-alpine3.20

RUN apk add --update --no-cache libxml2-dev libxslt-dev perl
RUN apk add --update --no-cache --virtual .build_deps build-base libffi-dev \
    && pip install parsedmarc \
    && cpan -i Email::Outlook::Message \
    && apk del .build_deps

RUN --mount=from=pkg-builder,source=/home/packager/packages/work,target=/packages \
    --mount=from=pkg-builder,source=/etc/apk/keys,target=/etc/apk/keys \
    apk add --no-cache --repository /packages \
    perl-email-outlook-message

COPY parsedmarc.ini /
COPY GeoLite2-Country.mmdb /usr/share/GeoIP/GeoLite2-Country.mmdb
