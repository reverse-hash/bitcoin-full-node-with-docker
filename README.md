<div align="center">
  <h1>Bitcoin full node with docker</h1>

  <img alt="Logo" src="./.doc/readme/logo.png" width="160"/>

  <p>
    <strong>A simple way to deploy your own node with Docker!</strong>
  </p>

  <p>
  <a href="https://github.com/reverse-hash/bitcoin-full-node-with-docker/actions/workflows/build.yml">
<img alt="" src="https://github.com/reverse-hash/bitcoin-full-node-with-docker/actions/workflows/build.yml/badge.svg"></a>
    <a href="./LICENSE.txt"><img alt="MIT" src="https://img.shields.io/badge/license-MIT-blue.svg"/></a>

  </p>

<strong><a href="#documentation">Documentation</a> </strong>
| <strong><a href="https://github.com/reverse-hash/bitcoin-full-node-with-docker/discussions">Support</a></strong>
| <strong><a href="./FAQ.md">FAQ</a></strong>

</div>

## About the project

Dockerized node to access on your LAN. On port 50002, <a href="https://github.com/romanz/electrs">Electrs</a> will be available to connect any Electrum compatible wallet and via HTTPS on port 3003, <a href="https://github.com/janoside/btc-rpc-explorer">BTC RPC Explorer</a> to explore the blockchain. The node will participate in the Bitcoin network by exchanging blocks with other nodes via Tor; and optionally, it will be accesible from anywhere also via Tor.

The following services are deployed:

| Service                                                                         | Version      | Base image         | Size    |
| ------------------------------------------------------------------------------- | ------------ | ------------------ | --------|
| <a href="https://gitlab.torproject.org/tpo/core/tor/">Tor</a>                   | 0.4.8.12     | debian:stable-slim | 83.2 MB |
| <a href="https://github.com/bitcoin/bitcoin">Bitcoin core</a>                   | 27.1         | debian:stable-slim | 79.7 MB |
| <a href="https://github.com/romanz/electrs">Electrum rust service</a>           | 0.10.5       | debian:stable-slim | 82.5 MB |
| <a href="https://github.com/janoside/btc-rpc-explorer">Bitcoin RPC Explorer</a> | 3.4.0        | node:22-slim       | 374 MB  |
| <a href="https://github.com/nginxinc/docker-nginx">NGINX</a>                    | stable-slim  | nginx:alpine-slim  | 11.5 MB |

## Documentation

<a href="#documentation"></a>

- <a href="./GETTING_STARTED.md">Getting started</a>
- <a href="./UPDATING_SERVICES.md">Updating services</a>

## Special thanks and attributions

- The current logo is a modification of the <a href="https://fontawesome.com/icons/docker">docker logo</a> from <a href="https://fontawesome.com">Font Awesome</a> under the (CC BY 4.0). The bitcoin logo has been added and colored orange.
- Kudos to Emmanuel Rosa for a an initial <a href="https://github.com/emmanuelrosa/bitcoin-onion-nodes">list of nodes for Tor</a>.
- Kudos to <a href="https://github.com/cozybear-dev">cozybear-dev</a> for a true multiarch Tor proxy and multiple improvements in documentation/security.
