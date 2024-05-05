# Getting started

- <a href="#prerequisites">Prerequisites</a>
- <a href="#setup-the-host-machine">Setup the host machine</a>
- <a href="#setup-the-multi-container-applications">Setup the multi-container applications</a>
- <a href="#running-services-for-the-first-time">Running services for the first time</a>
  - <a href="#tor">tor</a>
  - <a href="#bitcoind">bitcoind</a>
  - <a href="#electrs">electrs</a>
  - <a href="#btc-rpc-explorer">btc-rpc-explorer</a>
  - <a href="#nginx">nginx</a>
- <a href="#using-your-node">Using your node</a>
- <a href="#post-installation">Post-installation</a>
- <a href="#remote-access-to-your-node-via-tor-optional">Remote access to your node via tor (Optional)</a>

## Prerequisites

It's possible to run the node on a wide variety of systems, architectures, linux distributions and even in the cloud. We recommend using a dedicated computer with low power consumption and enough resources.

The most tested environments with which we have play with is:

- Operating system: Linux distro (Arch, Ubuntu, Debian...)
- CPU: > 1 Ghz
- Memory (RAM): > 2 GB
- Disk space: 600 GB

As we need to download the entire blockchain (480GB to this day) and it keeps growing, it's highly recommended to use some external device.

## Setup the host machine

Let's start from the point where we have a freshly installed Linux distribution, internet connection and a partition or hard disk connected with enough space. We also assume that the repository is cloned somewhere in your home folder and that we have already installed `docker` and `docker-compose`.

To make this guide as standardized as possible, we will use commands supported on most platforms. If for whatever reason it doesn't work, try to find the equivalent. But don't worry, there are not many and docker will do the rest.

Then create a new group with which we will control the read and write permissions between the local machine and the containers.

For this example, the name is `btcnode` but we can call it whatever we want.

```shell
$ sudo groupadd btcnode
```

Execute the following command to obtain the GID of the group. Make a note of it as we will need it later. For this example, the GID is `1099`.

```shell
$ getent group btcnode
# btcnode:x:1099:satoshi
```

Then add your host user to this group. For this example, the user is `satoshi`.

```shell
$ sudo usermod -a -G btcnode satoshi
```

Probably, you want to store the blockchain to some external device or directory outside the repository. So what you should do is copy the directories from the volumes folder to the desired location. For this example, the desired location is an empty USB hard drive mounted on `/mnt/hdd`.

```shell
$ sudo copy -R <repository>/volumes/* /mnt/hdd
```

Next, change the group and permission level for the volumes. In this way, all `btcnode` users will be able to read and write shared files on that folder:

```shell
$ sudo chgrp btcnode -R /mnt/hdd/*
$ sudo chmod 660 -R /mnt/hdd/*
$ sudo chmod +X -R /mnt/hdd/*
```

## Setup the multi-container applications

The container environment must be defined in a file `.env` in the repository root directory. Just make a copy of the sample file and edit it.

```shell
$ copy .env.example .env
```

It's crucial that the following parameters are properly configured or the environment will not work. The rest of the parameters can be left as they are.

```conf
# The GID we have previously noted.
SHARED_GID=1099

# The paths where to mount the volumes.
TOR_DATA=/mnt/hdd/tor
BITCOIN_DATA=/mnt/hdd/bitcoind
ELECTRS_DATA=/mnt/hdd/electrs
BTC_RPC_EXPLORER_DATA=/mnt/hdd/btcrpcexplorer
NGINX_DATA=/mnt/hdd/nginx
```

## Running services for the first time

For the first installation, we recommend starting the services one at a time. Take your time to verify by yourself the `Dockerfiles` and validate that the services are being deployed correctly.

It is also important because bitcoind will take a long time to synchronize, and if in the meantime, the rest of the containers that depend on it are continuously failing because the service is not available, these are resources that will make the process take even longer.

### tor

In most of the cases, tor has all the configuration you need. The only configurable parameter that is not enabled by default are the hidden services and we will review it later. In case you want to customize anything, the tor configuration file is located in `/mnt/hdd/tor/torrc.default`.

Run the following command `docker-compose up -d tor` to start the service and check the logs to be sure that the service is running properly.

For this example, we check the logs and everything looks good.

```shell
$ docker-compose up tor -d
$ docker ps | grep tor
# 06a96296854a   tor:12.0.1
$ docker logs -f 06a96296854a
# [notice] Tor 0.4.7.12 (git-f43a74975ed27d78) running on Linux with Libevent 2.1.12-stable, OpenSSL 1.1.1s, Zlib 1.2.11, Liblzma N/A, Libzstd N/A and Glibc 2.31 as libc.
# [notice] Tor can't help you if you use it wrong! Learn how to be safe at https://support.torproject.org/faq/staying-anonymous/
# [notice] Read configuration file "/home/tor/torrc.default".
# [warn] You specified a public address '0.0.0.0:9050' for SocksPort. Other people on the Internet might find your computer and use it as an open proxy. Please don't allow this unless you have a good reason.
# [notice] Opening Socks listener on 0.0.0.0:9050
# [notice] Opened Socks listener connection (ready) on 0.0.0.0:9050
# [notice] Bootstrapped 0% (starting): Starting
# ... Some more lines
# [notice] Bootstrapped 95% (circuit_create): Establishing a Tor circuit
# [notice] Bootstrapped 100% (done): Done
```

### bitcoind

The bitcoind configuration file is located in `/mnt/hdd/bitcoind/bitcoin.conf`. The default parameters are enough, except for one that depends on the memory of your local machine and the default password for communication between services, which you should generate yourself for security reasons.

The more cache bitcoind has, the faster it will be downloading and synchronizing the blocks. As a reference, you can set half of the machine memory expressed in MB. For example, if the node has 4GB of RAM, you can put 2048.

```conf
dbcache=2048
```

To generate your own passwords you can use the script available in the Bitcoin core repository:

```shell
cd /tmp
curl https://raw.githubusercontent.com/bitcoin/bitcoin/master/share/rpcauth/rpcauth.py -s -o rpcauth.py
python3 rpcauth.py btcrpcexplorer
#String to be appended to bitcoin.conf:
#rpcauth=btcrpcexplorer:0d46b997158b1396004ad5c742ca1dd5$7e98b6d4b00a62c16095038013a76a8ce71329ac195ddb1694308c552a1f8d93
#Your password:
#_PwADE_QHUCPE06hKNxaFbeWtKYB5DM7M6V43MKqDq0
python3 rpcauth.py electrs
#String to be appended to bitcoin.conf:
#rpcauth=electrs:8fa8d4bfee39fe3640a504c59b5cf99a$83023437b424aa81bf476393bf1767c4ae00e818edb58daef390e814ac9a8238
#Your password:
#Myg7ikhuvQjnf3AvgtB4xiUQjXcN5Nkt0E_fGCJbCMw
rm /tmp/rpcauth.py
```

Replace the two strings in the configuration file and take note of the passwords for the next steps.

This is the longest process and until it is finished we cannot continue. Your node will start synchronizing the entire blockchain with the rest of peers. This process can take days or even weeks. It's very difficult to give an estimation, because it depends on the capacity of your hardware, the state of the network, your connection speed... So be patient.

Run the following command `docker-compose up -d bitcoind` to start the service and check the logs to be sure that the services is running properly.

For this example, after a certain time, bitcoind was already synchronizing the 769944 block height (`height=769944`) and almost all the blockchain was already synchronized (`progress=0.999838`). At the time of writing this document, this was the last block mined, so the node was synchronized and we can move the next step.

```shell
$ docker-compose up bitcoind -d
$ docker ps | grep bitcoind
# 7cc89e57effb   bitcoind:24.0.1
$ docker logs -f 7cc89e57effb
# You should see it synchronizing blocks as follows:
#
# Bitcoin Core version v24.0.1 (release build)
# ... Some more lines
# net thread start
# msghand thread start
# opencon thread start
# addcon thread start
# ... At this point, it should be syncornizing blocks (height=X block number)
# UpdateTip: new best=0000000000000000000623fbc8f9cab4187f242ab5cdf062949a81d66c2be17a height=769942 version=0x26278000 log2_work=93.925313 tx=792669265 date='2023-01-02T00:28:38Z' progress=0.999837 cache=2.1MiB(15925txo)
# UpdateTip: new best=000000000000000000026ef7879a3fd57825a415c2e26ddf028c75f04087be58 height=769943 version=0x20400000 log2_work=93.925325 tx=792671631 date='2023-01-02T00:28:51Z' progress=0.999837 cache=4.0MiB(29419txo)
# UpdateTip: new best=00000000000000000006a28f25197b1dcc3eeab5d7dbb57b77c45cc38a3fa255 height=769944 version=0x20000000 log2_work=93.925337 tx=792672968 date='2023-01-02T00:35:48Z' progress=0.999838 cache=5.4MiB(41166txo)
```

### electrs

The electrs configuration file is located in `/mnt/hdd/electrs/electrs.conf`. The default parameters are enough, except for the password which must be replaced by the one generated earlier when configuring bitcoind.

Run the following command `docker-compose up -d electrs` to start the service and check the logs to be sure that the service is running properly.

For this example, we can see that electrs has already indexed the entire blockchain (`height=769944`, same block height as bitcoind).

```shell
$ docker ps | grep electrs
# 92ebe51900c9   electrs:0.9.12
$ docker logs -f 92ebe51900c9
# Starting electrs 0.9.12 on x86_64 linux with Config { network: Bitcoin, db_path: "/home/electrs/.electrs/db/bitcoin", daemon_dir: "/home/electrs/.bitcoin", daemon_auth: UserPass("electrs", "<sensitive>"), daemon_rpc_addr: 172.18.0.3:8332, daemon_p2p_addr: 172.18.0.3:8333, electrum_rpc_addr: 0.0.0.0:50001, monitoring_addr: 127.0.0.1:4224, wait_duration: 10s, jsonrpc_timeout: 15s, index_batch_size: 10, index_lookup_limit: None, reindex_last_blocks: 0, auto_reindex: true, ignore_mempool: false, sync_once: false, disable_electrum_rpc: false, server_banner: "Welcome to electrs 0.9.12 (Electrum Rust Server)!", signet_magic: 3652501241, args: [] }
# [2023-01-02T15:45:56.500Z INFO  electrs::server] serving Electrum RPC on 0.0.0.0:50001
# [2023-01-02T15:45:56.705Z INFO  electrs::db] "/home/electrs/.electrs/db/bitcoin": 163 SST files, 39.804026748 GB, 4.979395287 Grows
# [2023-01-02T17:17:50.099Z INFO  electrs::chain] chain updated: tip=000000000000000000021f6a20a394c43b89b13034e54ff3150e147a261a3b53, height=769944
```

### btc-rpc-explorer

The btc-rpc-explorer configuration file is located in `/mnt/hdd/btcrpcexplorer/btc-rpc-explorer.env`. The default parameters are also enough, except for the password which must be replaced by the one generated earlier when configuring bitcoind.

Run the following command `docker-compose up -d btcrpcexplorer` to start the service and check the logs to be sure that the service is running properly.

For this example, we can see that the service has beed started and it's connected to bitcoind (`RPC Connected: ... subversion=/Satoshi:24.0.1`)

```shell
$ docker-compose up -d btcrpcexplorer
$ docker ps | grep btcrpcexplorer
# c8b93a8b9410   btc-rpc-explorer:3.3.0
$ docker logs -f c8b93a8b9410
# > btc-rpc-explorer@3.3.0 start
# > node ./bin/www

# 2023-01-02T17:21:33.513Z btcexp:app Searching for config files...
# 2023-01-02T17:21:33.514Z btcexp:app Config file found at /home/btcrpcexplorer/.config/btc-rpc-explorer.env, loading...
# 2023-01-02T17:21:33.515Z btcexp:app Config file not found at /etc/btc-rpc-explorer/.env, continuing...
# 2023-01-02T17:21:33.515Z btcexp:app Config file not found at /opt/btc-rpc-explorer/.env, continuing...
# 2023-01-02T17:21:34.448Z btcexp:app Default cacheId '3.3.0'
# 2023-01-02T17:21:34.473Z btcexp:app Enabling view caching (performance will be improved but template edits will not be reflected)
# 2023-01-02T17:21:34.479Z btcexp:app Environment(development) - Node: v16.19.0, Platform: linux, Versions: {"node":"16.19.0","v8":"9.4.146.26-node.24","uv":"1.43.0","zlib":"1.2.11","brotli":"1.0.9","ares":"1.18.1","modules":"93","nghttp2":"1.47.0","napi":"8","llhttp":"6.0.10","openssl":"1.1.1s+quic","cldr":"41.0","icu":"71.1","tz":"2022f","unicode":"14.0","ngtcp2":"0.8.1","nghttp3":"0.7.0"}
# 2023-01-02T17:21:34.479Z btcexp:app No sourcecode version available, continuing to use default cacheId '3.3.0'
# 2023-01-02T17:21:34.479Z btcexp:app Starting BTC RPC Explorer, v3.3.0 at http://172.18.0.5:3002/
# 2023-01-02T17:21:34.479Z btcexp:app Connecting to RPC node at bitcoind:8332
# 2023-01-02T17:21:34.484Z btcexp:app Verifying RPC connection...
# 2023-01-02T17:21:34.487Z btcexp:app Loading mining pools config
# 2023-01-02T17:21:34.515Z btcexp:app RPC Connected: version=240001 subversion=/Satoshi:24.0.1/, parsedVersion(used for RPC versioning)=24.0.1, protocolversion=70016, chain=main, services=[NETWORK, WITNESS, NETWORK_LIMITED]
```

### nginx

The nginx configuration file is located in `/mnt/hdd/nginx/nginx.conf`. The default parameters are enough, but it will be necessary to create your our own SSL certificate to connect btc-rpc-explorer and electrs.

So, move to the nginx data volume folder and generate a self-signed certificate running the following command:

```shell
$ cd /mnt/hdd/nginx
$ openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out certificate.crt -keyout certificate.key
# you can simply press enter when you are prompted for an input value
```

Two files `certificate.crt` and `certificate.key` will be generated. It is important to use this name as they are specified in the nginx configuration file.

Finally, run the following command `docker-compose up -d nginx` to start the service and check the logs to be sure that the service is running properly.

For this example, we can see that the service has beed started and it's connected to bitcoind.

```shell
$ docker ps | grep nginx
# a811e7fd2455   nginx:alpine-slim 0.0.0.0:3003->3003/tcp, :::3003->3003/tcp, 80/tcp, 0.0.0.0:50002->50002/tcp, :::50002->50002/tcp
$ docker logs -f a811e7fd2455
#...
# /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
# /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
# /docker-entrypoint.sh: Configuration complete; ready for start up
```

## Using your node

If you have reached this point, you should be able to access btc-rpc-explorer and electrs through nginx.

- You can try to access to btc-rpc-explorer through the following URL: `https://your_node_ip:3003`.
- You can try to connect Sparrow or Electrum Wallet through the following URL: `https://your_node_ip:50002`

**Important**: The SSL certificate we generate is self-signed, this means that it's not signed by a recognized authority. It is normal if your browser indicates that it is not secure with a warning. This does not mean that the connection is not encrypted. It is possible to generate a certificate with a recognized authority by paying or through certain registration processes. However, this is outside the scope of this document.

## Post-installation

Once the node has been synchronized, we can change the bitcoind dbcache to the minimum. It is no longer necessary to have so much memory for this purpose.

Edit the bitcoind configuration file and change this parameter.

```conf
dbcache=360
```

Take this as an opportunity to restart the whole infrastructure and verify that we can shutdown and start everything without any problem.

```shell
$ docker-compose down
$ docker-compose up -d
```

## Remote access to your node via tor (Optional)

Tor allows you to expose services through a feature called hidden services. If we enable this, your node will expose services like btcrpcexplorer and electrs to the tor network, so you will be able to access them from anywhere if you have access to the tor network.

Our recommendation is to enable it only if you are going to use it very often or to enable it only for a period of time when you need it and then disable it again. It's just a matter of configuring some settings and restarting the tor service. Take this into consideration because the node will be accessible to anyone who discovers the service, and although the node does not store any private key, it is still a service exposed to attacks.

In case you want to do so, the tor configuration file is located in `/mnt/hdd/tor/torrc.default`. So open it with a text editor and uncomment the lines indicated in the file.

```conf
# Uncomment to expose Bitcoin Explorer as an Onion Service
HiddenServiceDir /home/tor/btc-rpc-explorer
HiddenServicePort 3003 nginx:3003

# Uncomment to expose Electrs as an Onion Service
HiddenServiceDir /home/tor/electrs
HiddenServicePort 50002 nginx:50002
```

Once the file has been edited, just restart tor.

```shell
$ docker-compose restart tor
```

Next, in the tor data volume you will find a folder for each hidden service. If you look inside, you will find a file with the hostname assigned.

For this example, the hostname assigned to the `btc-rpc-explorer` service is `vpu4f4pyyp4zruhllacgymj3qg67mgkwu5o37e3g2ad2un3rufew5xqd.onion` and is accessible from anywhere through the tor network.

```shell
$ cd /mnt/hdd/tor
$ ls
#  btc-rpc-explorer  electrs  torrc.default
$ cat btc-rpc-explorer/hostname
# vpu4f4pyyp4zruhllacgymj3qg67mgkwu5o37e3g2ad2un3rufew5xqd.onion
```

To revert this configuration, comment the lines again, and restart tor.
