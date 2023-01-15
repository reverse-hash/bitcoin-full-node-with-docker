FROM alpine:latest AS build
ARG BTC_RPC_EXPLORER_VERSION
RUN apk --update add wget \
  && wget https://github.com/janoside/btc-rpc-explorer/archive/refs/tags/v${BTC_RPC_EXPLORER_VERSION}.tar.gz -P /tmp \
  && tar -zxf /tmp/v${BTC_RPC_EXPLORER_VERSION}.tar.gz -C /opt \
  && mv /opt/btc-rpc-explorer-${BTC_RPC_EXPLORER_VERSION} /opt/btc-rpc-explorer \
  && chmod 755 -R /opt/btc-rpc-explorer


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
