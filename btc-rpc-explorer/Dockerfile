FROM node:16-alpine

ARG BTC_RPC_EXPLORER_VERSION

RUN apk --update add wget \
  && wget https://github.com/janoside/btc-rpc-explorer/archive/refs/tags/v${BTC_RPC_EXPLORER_VERSION}.tar.gz -P /tmp \
  && tar -zxf /tmp/v${BTC_RPC_EXPLORER_VERSION}.tar.gz -C /opt \
  && mv /opt/btc-rpc-explorer-${BTC_RPC_EXPLORER_VERSION} /opt/btc-rpc-explorer \
  && chmod 755 -R /opt/btc-rpc-explorer \
  && cd /opt/btc-rpc-explorer \
  && npm install
  
RUN adduser -D btcrpcexplorer \
  && mkdir /home/btcrpcexplorer/.config \
  && chown btcrpcexplorer:btcrpcexplorer /home/btcrpcexplorer/.config
  
COPY --chown=btcrpcexplorer:btcrpcexplorer btc-rpc-explorer.env /home/btcrpcexplorer/.config

USER btcrpcexplorer

CMD cd /opt/btc-rpc-explorer && npm start
