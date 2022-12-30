FROM debian:stable-slim

ARG BITCOIN_VERSION
ARG BITCOIN_PLATFORM
ARG USER_ID
ARG GROUP_ID

ENV PATH=/opt/bitcoin-${BITCOIN_VERSION}/bin:$PATH

RUN apt-get update -y \
  && apt-get install gpg wget -y \
  && apt-get clean
  
RUN mkdir /tmp/bitcoin-core \ 
  && cd /tmp/bitcoin-core \
  && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS \
  && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/SHA256SUMS.asc \
  && wget https://raw.githubusercontent.com/bitcoin/bitcoin/master/contrib/builder-keys/keys.txt \
  && while read k; do gpg --keyserver hkps://keys.openpgp.org --recv-keys $(echo "$k" | awk '{print $1}'); done < keys.txt \
  && if [ "3" -lt $(gpg --verify SHA256SUMS.asc 2>&1 >/dev/null | grep 'Good Signature' -c) ]; then exit 1; fi \
  && wget https://bitcoincore.org/bin/bitcoin-core-${BITCOIN_VERSION}/bitcoin-${BITCOIN_VERSION}-${BITCOIN_PLATFORM}.tar.gz \
  && sha256sum --ignore-missing --check SHA256SUMS
  
RUN tar -xzf /tmp/bitcoin-core/*.tar.gz -C /opt \
  && groupadd bitcoin -g ${GROUP_ID} \
  && useradd -u ${USER_ID} -g ${GROUP_ID} -m bitcoin \
  && mkdir /home/bitcoin/.bitcoin \
  && chown bitcoin:bitcoin /home/bitcoin/.bitcoin
  
COPY --chown=bitcoin:bitcoin bitcoin.conf /home/bitcoin/.bitcoin/bitcoin.conf
  
RUN apt-get remove gpg wget -y \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /opt/bitcoin-${BITCOIN_VERSION}/bin/bitcoin-qt

USER bitcoin

CMD bitcoind
