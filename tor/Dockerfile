FROM alpine:latest AS build
ARG TOR_VERSION
ARG TOR_PLATFORM
RUN apk update \
  && apk add --no-cache dirmngr gpg gpg-agent wget \
  && cd /tmp \
  && wget https://www.torproject.org/dist/torbrowser/${TOR_VERSION}/tor-expert-bundle-${TOR_VERSION}-${TOR_PLATFORM}.tar.gz.asc \
  && wget https://www.torproject.org/dist/torbrowser/${TOR_VERSION}/tor-expert-bundle-${TOR_VERSION}-${TOR_PLATFORM}.tar.gz \
  && gpg --auto-key-locate nodefault,wkd --locate-keys torbrowser@torproject.org \
  && gpg --verify tor-expert-bundle-${TOR_VERSION}-${TOR_PLATFORM}.tar.gz.asc \
  && if [ "1" -lt $(gpg --verify SHA256SUMS.asc 2>&1 >/dev/null | grep 'Good Signature' -c) ]; then exit 1; fi \
  && tar -xzf *.tar.gz -C . \
  && rm -rf tor/debug tor/data tor/pluggable_transports \
  && chmod -R 755 tor \
  && apk del dirmngr gpg gpg-agent wget


FROM debian:stable-slim
ARG GROUP_ID
COPY --from=build /tmp/tor /opt/tor
ENV PATH=$PATH:/opt/tor
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/tor
RUN addgroup --gid ${GROUP_ID} tor \
  && adduser --disabled-password --gecos "" --ingroup tor tor

EXPOSE 9050

USER tor

CMD tor -f ~/torrc.default
