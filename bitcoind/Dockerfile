FROM debian:stable-slim AS build
ARG BITCOIN_VERSION
RUN apt-get update \
  && apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils libevent-dev libboost-dev curl python3 byacc git wget -y \
  && cd /tmp \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/achow101.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/fanquake.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/guggero.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/hebasto.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/theStack.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/vertiond.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/willyko.gpg \
  && gpg --import *.gpg \
  && git clone -b v${BITCOIN_VERSION} https://github.com/bitcoin/bitcoin.git \
  && cd bitcoin \
  && git verify-tag v${BITCOIN_VERSION} \
  && cd depends \
  && make HOST=$(gcc -dumpmachine) NO_QT=1 NO_ZMQ=1 NO_WALLET=1 NO_BDB=1 NO_SQLITE=1 \
  && cd .. \
  && ./autogen.sh \
  && ./configure \
    --enable-glibc-back-compat \
    --prefix=$PWD/depends/$(gcc -dumpmachine) LDFLAGS="-static-libstdc++" \
    --disable-bench \
    --disable-man \
    --disable-tests \
    --disable-wallet \
    --with-qrencode=no \
    --enable-fuzz-binary=no \
    --without-gui \
  && make \
  && apt-get remove build-essential libtool autotools-dev automake pkg-config bsdmainutils libevent-dev libboost-dev curl python3 byacc git wget -y


FROM debian:stable-slim
ARG GROUP_ID
COPY --from=build /tmp/bitcoin/src/bitcoind /usr/local/bin/bitcoind
RUN addgroup --gid ${GROUP_ID} bitcoin \
  && adduser --disabled-password --gecos "" --ingroup bitcoin bitcoin \
  && mkdir /home/bitcoin/.bitcoin \
  && chown bitcoin:bitcoin /home/bitcoin/.bitcoin \
  # TODO: Remove when possible. We have to update glibc because when compiling we are using a diff glibc version (2.32 vs 2.31)
  && apt-get update \
  && apt-get install libc6 -y

USER bitcoin

CMD bitcoind
