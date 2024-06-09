FROM debian:stable-slim AS build
ARG BITCOIN_VERSION
RUN apt-get update \
  && apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils libevent-dev libboost-dev curl python3 byacc git wget -y \
  && cd /tmp \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/achow101.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/fanquake.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/glozow.gpg \
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
    --prefix=$PWD/depends/$(gcc -dumpmachine) LDFLAGS="-static-libstdc++" \
    --disable-bench \
    --disable-cli \
    --disable-debug \
    --disable-man \
    --disable-tests \
    --disable-upnp \
    --disable-wallet \
    --disable-zmq \
    --with-qrencode=no \
    --enable-fuzz-binary=no \
    --enable-strip \
    --without-gui \
    --without-bdb \
    --with-sqlite=no \    
  && make \
  && gzexe src/bitcoind \
  && apt-get remove build-essential libtool autotools-dev automake pkg-config bsdmainutils libevent-dev libboost-dev curl python3 byacc git wget -y


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
