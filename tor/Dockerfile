FROM debian:stable-slim

ARG TOR_VERSION
ARG TOR_PLATFORM

ENV PATH=/opt/tor:$PATH
ENV LD_LIBRARY_PATH=/opt/tor:$LD_LIBRARY_PATH

RUN apt-get update -y \
  && apt-get install gpg wget -y \
  && apt-get clean

RUN mkdir /tmp/tor \
  && cd /tmp/tor \
  && wget https://www.torproject.org/dist/torbrowser/${TOR_VERSION}/tor-expert-bundle-${TOR_VERSION}-${TOR_PLATFORM}.tar.gz.asc \
  && wget https://www.torproject.org/dist/torbrowser/${TOR_VERSION}/tor-expert-bundle-${TOR_VERSION}-${TOR_PLATFORM}.tar.gz \
  && gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org \
  && gpg --verify tor-expert-bundle-${TOR_VERSION}-${TOR_PLATFORM}.tar.gz.asc \
  && if [ "1" -lt $(gpg --verify SHA256SUMS.asc 2>&1 >/dev/null | grep 'Good Signature' -c) ]; then exit 1; fi

RUN tar -xzf /tmp/tor/*.tar.gz -C /opt \
  && useradd -m tor \
  && chmod -R 755 /opt/tor

COPY --chown=tor:tor torrc.default /home/tor/torrc.default
  
RUN apt-get remove gpg wget -y \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /opt/tor/debug /opt/tor/data

EXPOSE 9050

USER tor

CMD tor -f ~/torrc.default
