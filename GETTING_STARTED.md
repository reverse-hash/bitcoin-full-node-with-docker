# Getting started

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
btcnode:x:1099:satoshi
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

The container environment must be defined in a file `.env` in the repository root directory. Just  make a copy of the sample file and edit it.

```shell
$ copy .env.example .env
```

It's crucial that the following parameters are properly configured or the environment will not work.

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

## Configure tor

In most cases, tor has all the configuration you need. The only configurable parameter that is not enabled by default is the hidden services and we will review it later.

In case you want to customize anything, the tor configuration file is located in `/mnt/hdd/tor/torrc.default`.

## Starting tor

Run the following command `docker-compose up tor -d` to start service and check the logs to be sure that the services is running properly.

For this example, we check the logs and everything looks good.

```shell
$ docker-compose up tor -d 
$ docker ps | grep tor
06a96296854a   tor:12.0.1
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

## Configure bitcoind

The bitcoind configuration file is located in `/mnt/hdd/bitcoind/bitcoin.conf`. The default parameters are enough, except for one that depends on the memory of your local machine.

The more cache bitcoind has, the faster it will be downloading and synchronizing the blocks. As a reference, you can set half of the machine memory expressed in MB. For example, if the node has 4GB of RAM, you can put 2048.

```conf
dbcache=2048
```
## Starting bitcoind

This is the longest process and until it is finished we cannot continue. Our node will start synchronizing the entire blockchain with the rest of peers. This process can take days or even weeks. It's very difficult to give an estimate, because it depends on the capacity of your hardware, the state of the network, your connection speed... So be patient.

Run the following command `docker-compose up bitcoind -d` to start service and check the logs to be sure that the services is running properly.

For this example, after a certain time, bitcoind was already synchronizing the 769944 block height (`height=769944`)  and almost all the blockchain was already synchronized (`progress=0.999838`). At the time of this writing, this was the last block mined, so the node was synchronized and we can move the next step.

```
$ docker-compose up bitcoind -d
$ docker ps | grep bitcoind
7cc89e57effb   bitcoind:24.0.1
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
UpdateTip: new best=0000000000000000000623fbc8f9cab4187f242ab5cdf062949a81d66c2be17a height=769942 version=0x26278000 log2_work=93.925313 tx=792669265 date='2023-01-02T00:28:38Z' progress=0.999837 cache=2.1MiB(15925txo)
UpdateTip: new best=000000000000000000026ef7879a3fd57825a415c2e26ddf028c75f04087be58 height=769943 version=0x20400000 log2_work=93.925325 tx=792671631 date='2023-01-02T00:28:51Z' progress=0.999837 cache=4.0MiB(29419txo)
UpdateTip: new best=00000000000000000006a28f25197b1dcc3eeab5d7dbb57b77c45cc38a3fa255 height=769944 version=0x20000000 log2_work=93.925337 tx=792672968 date='2023-01-02T00:35:48Z' progress=0.999838 cache=5.4MiB(41166txo)
```

## Configure btc-rpc-explorer

The btc-rpc-explorer configuration file is located in `/mnt/hdd/btcrpcexplorer/btc-rpc-explorer.env`. The default parameters are enough, but in case you're interested, here it is.

The more cache bitcoind has, the faster it will be downloading and synchronizing the blocks. As a reference, you can set half of the machine memory expressed in MB. For example, if the node has 4GB of RAM, you can put 2048.


## Configure nginx

It's also necessary to create your our own SSL certificate to be able to connect to nginx, which works as a reverse proxy to access `btc-rpc-explorer` and `electrs`.

Go to the directory where you have mounted the nginx data volume and run the following command:

```shell
$ cd /mnt/hdd/nginx
$ openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out certificate.crt -keyout certificate.key
# you can simply press enter when you are prompted for an input value
```



Create self-signed certification
```
$ openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out btcnode.crt -keyout btcnode.key
# all enter
```

## Remote access to your node via tor (Optional)
  
If we enable this feature, your node will expose services like btcrpcexplorer and electrs to the tor network, so you will be able to access them from anywhere if you have access to the tor network.

Our recommendation is to enable it only if you are going to use it. Another option is to enable it only for a period of time when you need it and then disable it again. It's just a matter of configuring some settings and restarting the tor service.

In case you want to do so, the tor configuration file is located in `/mnt/hdd/tor/torrc.default`. So open it with a text editor and uncomment the lines indicated in the file.





```shell
$ nano /mnt/hdd/tor/torrc.default
# Uncomment to expose Bitcoin Explorer as an Onion Service
HiddenServiceDir /home/tor/btc-rpc-explorer
HiddenServicePort 3003 nginx:3003

# Uncomment to expose Electrs as an Onion Service
HiddenServiceDir /home/tor/electrs
HiddenServicePort 50002 nginx:50002
```

Once the file has been edited, just restart tor.

```
$ cd <repository>
$ docker-compose restart tor
```

Once restarted, in the tor data volume you will find a folder for hidden service. If you look inside, in the hostname file you will fine hostname assigned to the service.

For this example, the hostname assigned to the `btc-rpc-explorer` service is `vpu4f4pyyp4zruhllacgymj3qg67mgkwu5o37e3g2ad2un3rufew5xqd.onion` and is accessible from anywhere through the tor network.

```
$ cd /mnt/hdd/tor
$ ls
btc-rpc-explorer  electrs  torrc.default
$ cat btc-rpc-explorer/hostname
vpu4f4pyyp4zruhllacgymj3qg67mgkwu5o37e3g2ad2un3rufew5xqd.onion
```

To revert this configuration, comment the lines again, and restart tor.
