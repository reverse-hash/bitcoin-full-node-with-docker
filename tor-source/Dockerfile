# Largely based on https://raw.githubusercontent.com/cmehay/docker-tor-hidden-service/master/Dockerfile
FROM python:3.10-alpine
ARG TOR_VERSION
ARG TORSOCKS_VERSION
ENV POETRY_VIRTUALENVS_CREATE=false

RUN apk add --no-cache git bind-tools cargo libevent-dev openssl-dev gnupg gcc make automake ca-certificates autoconf musl-dev coreutils libffi-dev zlib-dev && \
    mkdir -p /usr/local/src/ /var/lib/tor/ && \
    git clone https://git.torproject.org/tor.git /usr/local/src/tor && \
    cd /usr/local/src/tor && \
    TOR_VERSION=${tor_version=$(git tag | grep -oE '[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$' | sort -V | tail -1)} && \
    git checkout tor-$TOR_VERSION && \
    ./autogen.sh && \
    ./configure \
    --disable-asciidoc \
    --sysconfdir=/etc \
    --disable-unittests && \
    make && make install && \
    cd .. && \
    rm -rf tor && \
    pip3 install --upgrade pip poetry && \
    apk del git libevent-dev openssl-dev gnupg cargo make automake autoconf musl-dev coreutils libffi-dev && \
    apk add --no-cache libevent openssl

RUN apk add --no-cache git gcc make automake autoconf musl-dev libtool && \
    git clone https://git.torproject.org/torsocks.git /usr/local/src/torsocks && \
    cd /usr/local/src/torsocks && \
    TORSOCKS_VERSION=${torsocks_version=$(git tag | grep -oE 'v[0-9]+\.[0-9]+\.[0-9]+$' | sort -V | tail -1)} && \
    git checkout $TORSOCKS_VERSION && \
    ./autogen.sh && \
    ./configure && \
    make && make install && \
    cd .. && \
    rm -rf torsocks && \
    apk del git gcc make automake autoconf musl-dev libtool

ARG GROUP_ID

ENV PATH=$PATH:/var/lib/tor
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/var/lib/tor

RUN addgroup --gid ${GROUP_ID} tor \
  && adduser --disabled-password --gecos "" --ingroup tor tor

USER tor

CMD tor -f ~/torrc.default