FROM debian:stable-slim AS build
ARG BITCOIN_VERSION
RUN apt-get update \
  && apt-get install automake autotools-dev bsdmainutils build-essential curl git libtool pkg-config -y \
  && cd /tmp \
  && curl \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/achow101.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/fanquake.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/glozow.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/guggero.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/hebasto.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/theStack.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/vertiond.gpg \
    -O https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/willyko.gpg \
  && gpg --import *.gpg \
  && git clone -b v${BITCOIN_VERSION} https://github.com/bitcoin/bitcoin.git \
  && cd bitcoin \
  && git verify-tag v${BITCOIN_VERSION} \
  && cd depends \
  && make HOST=$(gcc -dumpmachine) NO_BDB=1 NO_QT=1 NO_QR=1 NO_SQLITE=1 NO_NATPMP=1 NO_UPNP=1 NO_ZMQ=1 NO_USDT=1 NO_WALLET=1 NO_HARDEN=1 \
  && cd .. \
  && ./autogen.sh \
  && ./configure \
    --prefix=$PWD/depends/$(gcc -dumpmachine) LDFLAGS="-static-libstdc++ -g0" \
    --disable-bench \
    --disable-cli \
    --disable-external-signer \
    --disable-fuzz-binary \
    --disable-man \
    --disable-tests \
    --enable-reduce-exports \
    --without-utils \
  && make \
  && gzexe src/bitcoind \
  && apt-get remove automake autotools-dev bsdmainutils build-essential curl git libtool pkg-config -y


FROM debian:stable-slim
ARG BITCOIN_USER_ID
ARG GROUP_ID
COPY --from=build /tmp/bitcoin/src/bitcoind /usr/local/bin/bitcoind
RUN addgroup --gid ${GROUP_ID} bitcoin \
  && adduser -u ${BITCOIN_USER_ID} --disabled-password --gecos "" --ingroup bitcoin bitcoin \
  && mkdir /home/bitcoin/.bitcoin \
  && chown bitcoin:bitcoin /home/bitcoin/.bitcoin

USER bitcoin

CMD bitcoind
