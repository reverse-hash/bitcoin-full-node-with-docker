<div align="center">
  <h1>Bitcoin full node with docker</h1>

  <img alt="Logo" src="./.doc/readme/logo.png" width="220"/>

  <p>
    <strong>A simple, lightweight, secure, private and verifiable way to deploy your own node with Docker!</strong>
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

There are many alternatives to set up your own node, either by installing everything from scratch, with preconfigured images or even buying one ready to use. Some options seem too crafty and difficult to maintain, and others too customized that add extra complexity that makes auditing the node very difficult.

Since no option was working for me, I decided to create this project and focus on the following points:

- Simplicity: in the sense that anyone can understand the node arquitecture in a matter of minutes.
- Lightweight: it can run on a low-resource computer.
- Secure: it not expose personal data or funds to a possible attacker. 
- Private: communications are encrypted and done anonymously.
- Verifiable: the build process must be open and simple, so that anyone with a minimum tech-background can verify it.


So here we are and this is what we have:

- A system to deploy and keep a Bitcoin node up to date in an automated way.
- No third parties involved. All software is downloaded and verified from official sites.
- The build process is based on `Docker` and a few `Dockerfiles`. No pre-builded images.
- Everything is built on your local machine and you will verify it.

## What to expect

As Bob, you will have a dockerized node that you can access on your LAN. On port 50002 you will have an <a href="https://github.com/romanz/electrs">Electrs</a> service to connect <a href="https://github.com/spesmilo/electrum">Electrum</a>, <a href="https://github.com/sparrowwallet/sparrow">Sparrow</a>, <a href="https://github.com/bluewallet/bluewallet">Bluewallet</a> or any Electrum compatible wallet to check your balances and/or broadcast any transaction privately. In addition, via HTTPS on port 3003, you will have access to <a href="https://github.com/janoside/btc-rpc-explorer">BTC RPC Explorer</a> to explore the blockchain privately.

Your node will participate in the Bitcoin network exchanging blocks with other nodes through Tor; and optionally, it will be accesible from anywhere also through Tor.

<div style="border-radius:20px;padding:5px;background-color:#0d1117">
<img src=".doc/readme/diagram.svg"/>
</div>

The following services are deployed:

| Container | Service | Base image | Size |
| --- | --- | --- | --- |
| tor | Tor 0.4.7.13 | debian:stable-slim | 99.6 MB |
| bitcoind | Bitcoin core daemon 25.0 | debian:stable-slim | 96.2 MB |
| electrs | Electrum rust service 0.9.13 | debian:stable-slim | 101 MB |
| btcrpcexplorer | Bitcoin explorer 3.0.3 | node:16-slim | 251 MB |
| nginx | NGINX stable | nginx:alpine-slim | 11.5 MB |


## Documentation
<a href="#documentation"></a>

- <a href="./GETTING_STARTED.md">Getting started</a>
- <a href="./UPDATING_SERVICES.md">Updating services</a>
- Extra (WIP)

## Special thanks and attributions

- The current logo is a modification of the <a href="https://fontawesome.com/icons/docker">docker logo</a> from <a href="https://fontawesome.com">Font Awesome</a> under the (CC BY 4.0). The bitcoin logo has been added and colored orange.
- Kudos to Emmanuel Rosa for a an initial <a href="https://github.com/emmanuelrosa/bitcoin-onion-nodes">list of nodes for Tor</a>.
- Kudos to <a href="https://github.com/cozybear-dev">cozybear-dev</a> for a true multiarch Tor proxy and multiple improvements in documentation/security.
