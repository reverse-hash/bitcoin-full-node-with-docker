FROM alpine:3.18 AS build
ARG BTC_RPC_EXPLORER_VERSION
RUN apk add --update --no-cache git gpg gpg-agent nodejs npm python3 wget \
  && cd /opt \
  && wget -O - https://keybase.io/danjanosik/pgp_keys.asc | gpg --import \
  && git clone --branch v${BTC_RPC_EXPLORER_VERSION} https://github.com/janoside/btc-rpc-explorer.git \
  && cd btc-rpc-explorer \
#  && git verify-tag v${BTC_RPC_EXPLORER_VERSION} \ FIXME: not working. Requested to btc-rpc-explorer developers
  && npm install \
  && chmod 755 -R . \
  && apk del git gpg gpg-agent nodejs npm python3 wget


FROM node:22-slim
ARG BTC_RPC_EXPLORER_USER_ID
ARG GROUP_ID
COPY --from=build /opt/btc-rpc-explorer /opt/btc-rpc-explorer
RUN addgroup --gid $GROUP_ID btcrpcexplorer \
  && adduser -u ${BTC_RPC_EXPLORER_USER_ID} --disabled-password --gecos "" --ingroup btcrpcexplorer btcrpcexplorer \
  && mkdir /home/btcrpcexplorer/.config \
  && chown btcrpcexplorer:btcrpcexplorer /home/btcrpcexplorer/.config \
  && cd /opt/btc-rpc-explorer

USER btcrpcexplorer

CMD cd /opt/btc-rpc-explorer && npm start
