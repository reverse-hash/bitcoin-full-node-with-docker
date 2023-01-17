# Installation

## Setup the host machine

Create a new group to control file sharing permissions between the local machine and the containers.

```shell
$ sudo groupadd btcnode
```

Then add your local user to this group. For this example, the main user on the local machine is satoshi.

```
$ sudo usermod -a -G btcnode satoshi
```

// TODO explain for what volumes, where to por it, etc.
Change the ownership and permission level. Note that the volume directory can be moved to any other directory. In fact, it is usually on an external device, such as a hard disk, SD card or pendrive. We will specify the correct path to the volumes in the next step. The correct path of the volumes will be specified later.

```
$ sudo chown satoshi:btcnode -R /your/custom/path/volumes/*
$ sudo chmod 660 -R /your/custom/path/volumes/*
$ sudo chmod +X -R /your/custom/path/volumes/*
```

Next, edit the `.env` file in the root of the repository. The file name starts with a dot, so it may not be visible. Edit it and adjust the following parameters.


```
$ getent group btcnode
btcnode:x:1099:satoshi
```

```
# The group identifier we created previously.Check the documentation to know how to obtain this identifier.
SHARED_GID=1099

# The paths where to mount the volumes. You *must* adjust these values to your system and map them to the correct paths:
BITCOIN_DATA=/mnt/hdd/bitcoin-core
ELECTRS_DATA=/mnt/hdd/electrs
BTC_RPC_EXPLORER_DATA=/mnt/hdd/btcrpcexplorer

```

```

$ docker-compose up tor -d 
$ docker ps # List existing containers
$ docker logs -f <tor container id> # Verify that tor is running correctly
# You should see something like this:
#
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


```
$ docker-compose up bitcoind -d
$ docker ps # List existing containers
$ docker logs -f <bitcoind container id> # Verify that bitcoind is running correctly
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

Create self-signed certification
```
$ openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out btcnode.crt -keyout btcnode.key
# all enter
```




Tools:

https://jlopp.github.io/bitcoin-core-config-generator/
# We can generate this value at https://jlopp.github.io/bitcoin-core-rpc-auth-generator.


https://github.com/emmanuelrosa/bitcoin-onion-nodes
