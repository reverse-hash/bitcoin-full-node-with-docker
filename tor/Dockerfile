FROM debian:stable-slim AS build
ARG TOR_VERSION
RUN apt-get update \
  && apt-get install build-essential automake libevent-dev libssl-dev zlib1g-dev git -y \
  && cd /tmp \
  && git clone -b tor-${TOR_VERSION} https://git.torproject.org/tor.git \
  && cd tor \
  && ./autogen.sh \
  && ln -s $(dpkg -L libevent-dev | grep libevent.a) /tmp/libevent.a \
  && ./configure \
    --disable-asciidoc \
    --sysconfdir=/etc \
    --disable-unittests \
    -enable-static-tor \
    --with-libevent-dir=$(dirname $(dpkg -L libevent-dev | grep libevent.a)) \
    --with-openssl-dir=$(dirname $(dpkg -L libssl-dev | grep libssl.a)) \
    --with-zlib-dir=$(dirname $(dpkg -L zlib1g-dev | grep libz.a)) \
  && make \
  && apt-get remove build-essential automake libevent-dev libssl-dev zlib1g-dev git -y


FROM debian:stable-slim
ARG GROUP_ID
COPY --from=build /tmp/tor/src/app/tor /opt/tor/tor
ENV PATH=$PATH:/opt/tor
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/tor
RUN addgroup --gid ${GROUP_ID} tor \
  && adduser --disabled-password --gecos "" --ingroup tor tor

USER tor

CMD tor -f ~/torrc.default
