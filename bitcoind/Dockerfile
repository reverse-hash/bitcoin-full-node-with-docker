FROM alpine:latest AS build
ARG BITCOIN_VERSION
ARG BITCOIN_PLATFORM
RUN apk update \
  && apk add --no-cache gpg gpg-agent wget \
  && cd /tmp \
  && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS \
  && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS.asc \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/achow101.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/fanquake.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/guggero.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/hebasto.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/theStack.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/vertiond.gpg \
  && wget https://raw.githubusercontent.com/bitcoin-core/guix.sigs/main/builder-keys/willyko.gpg \
  && gpg --import *.gpg \
  && if [ "3" -lt $(gpg --verify SHA256SUMS.asc 2>&1 >/dev/null | grep 'Good Signature' -c) ]; then exit 1; fi \
  && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/bitcoin-${BITCOIN_VERSION}-${BITCOIN_PLATFORM}.tar.gz \
  && cat SHA256SUMS | grep bitcoin-${BITCOIN_VERSION}-${BITCOIN_PLATFORM}.tar.gz | sha256sum -c \
  && tar --strip-component 2 -xzf bitcoin-*.tar.gz bitcoin-${BITCOIN_VERSION}/bin/bitcoind \
  && rm SHA256SUMS* *.gpg *.tar.gz \
  && apk del gpg gpg-agent wget


FROM debian:stable-slim
ARG GROUP_ID
COPY --from=build /tmp/bitcoind /usr/local/bin/bitcoind
RUN addgroup --gid ${GROUP_ID} bitcoin \
  && adduser --disabled-password --gecos "" --ingroup bitcoin bitcoin \
  && mkdir /home/bitcoin/.bitcoin \
  && chown bitcoin:bitcoin /home/bitcoin/.bitcoin

USER bitcoin

CMD bitcoind
