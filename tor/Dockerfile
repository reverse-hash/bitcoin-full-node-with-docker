FROM debian:stable-slim AS build
ARG TOR_VERSION
RUN apt-get update \
  && apt-get install build-essential automake libevent-dev libssl-dev zlib1g-dev git wget -y \
  && cd /tmp \
  # One or many of these keys can sign the releases. Verificable in https://gitlab.torproject.org/dgoulet/tor
  && wget https://keys.openpgp.org/vks/v1/by-fingerprint/1C1BC007A9F607AA8152C040BEA7B180B1491921 \
  && wget https://keys.openpgp.org/vks/v1/by-fingerprint/B74417EDDF22AC9F9E90F49142E86A2A11F48D36 \
  && wget https://keys.openpgp.org/vks/v1/by-fingerprint/2133BC600AB133E1D826D173FE43009C4607B1FB \
  && gpg --import * \
  && git clone -b tor-${TOR_VERSION} https://git.torproject.org/tor.git \
  && cd tor \
  && git verify-tag tor-${TOR_VERSION} \
  && ./autogen.sh \
  && ./configure \
    --disable-asciidoc \
    --sysconfdir=/etc \
    --disable-unittests \
    -enable-static-tor \
    --with-libevent-dir=$(dirname $(dpkg -L libevent-dev | grep libevent.a)) \
    --with-openssl-dir=$(dirname $(dpkg -L libssl-dev | grep libssl.a)) \
    --with-zlib-dir=$(dirname $(dpkg -L zlib1g-dev | grep libz.a)) \
  && make \
  && gzexe src/app/tor \
  && apt-get remove build-essential automake libevent-dev libssl-dev zlib1g-dev git wget -y


FROM debian:stable-slim
ARG TOR_USER_ID
ARG GROUP_ID
COPY --from=build /tmp/tor/src/app/tor /usr/local/bin/tor
RUN addgroup -u ${TOR_USER_ID} --gid ${GROUP_ID} tor \
  && adduser --disabled-password --gecos "" --ingroup tor tor

USER tor

CMD tor -f ~/torrc.default
