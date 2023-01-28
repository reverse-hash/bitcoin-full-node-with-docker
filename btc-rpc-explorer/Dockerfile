FROM alpine:latest AS build
ARG BTC_RPC_EXPLORER_VERSION
RUN apk --update add curl git gpg gpg-agent \
  && cd /opt \
  && curl https://keybase.io/danjanosik/pgp_keys.asc | gpg --import \
  && git clone --branch v${BTC_RPC_EXPLORER_VERSION} https://github.com/janoside/btc-rpc-explorer.git \
  && cd btc-rpc-explorer \
#  && git verify-tag v${BTC_RPC_EXPLORER_VERSION} \ FIXME: not working. Requested to btc-rpc-explorer developers
  && chmod 755 -R .


FROM node:16-slim
ARG GROUP_ID
COPY --from=build /opt/btc-rpc-explorer /opt/btc-rpc-explorer
RUN addgroup --gid $GROUP_ID btcrpcexplorer \
  && adduser --disabled-password --gecos "" --ingroup btcrpcexplorer btcrpcexplorer \
  && mkdir /home/btcrpcexplorer/.config \
  && chown btcrpcexplorer:btcrpcexplorer /home/btcrpcexplorer/.config \
  && cd /opt/btc-rpc-explorer \
  && npm install

USER btcrpcexplorer

CMD cd /opt/btc-rpc-explorer && npm start
