---
title: "Discovery , Bitcoin and Litecoin Configuration"
layout: single
permalink: /b4v10/DiscoveryConf
sidebar:
  nav: "overview"
toc: true
---

## Discovery
---------

Following secions are described under version 0.8, sample config file
can be reference from
[here](https://github.com/bitmark-inc/discovery/blob/master/discovery.conf.sample).

Discovery is used to find payment info, currently bitmarkd only support
crypto currenty of Bitcoin and Litecoin.

Overall flow as follows (arrow means service regristration direction):

```
                pub
            ---------->
   bitmarkd             discovery -----------> btc/ltc
            <----------               sub
                rep
```

discovery receives crypto currency by channel `sub`, notify `bitmarkd`
by `pub`, reply bitmarkd command by `rep`.

`pub_endpoint` and `rep_endpoint` are used to specify which IP and port
to communicate with `bitmarkd`. Please be noted that these two values
must be same as configuration in `bitmarkd`.

Sample Config:

``` shell
pub_endpoint = "127.0.0.1:5566"
rep_endpoint = "127.0.0.1:5567"
```

### Currency section

This part is used to denote supported crypto currency (bitcoin and
litecoin) settings.

1.  Bitcoin

    The `url` is for bitcoin rpc connection port.

    The `cached_block_count` is to specify how many blocks will be
    cached at most.

    The `sub_endpoint` is to specify bitcoin zero MQ IP and port. The
    value of this part should be same in bitcoin configuration file.

    Sample config:

    ``` shell
    bitcoin {
        url = "http://127.0.0.1:183333"
        sub_endpoint = "tcp://127.0.0.1:18332"
        cached_block_count = 500
    }
    ```

2.  Litecoin

    This part has similar fields as previous one. Sample config:

    ``` shell
    litecoin {
        url = "http://127.0.0.1:19335"
        sub_endpoint = "tcp://127.0.0.1:19332"
        cached_block_count = 1000
    }
    ```

### Logging section

This part is for logging program behavior, either for debug usage or
monitor status.

The `directory` is to denote where log file will be saved.

The `file` is used to specify log file name.

The `size` is used to specify maximum size of log file, be noted that
this value cannot be set less than 20000.

The `count` is used to denote how many files will be preserved by log
rotating, be noted that this value cannot be set less than 10.

The `levels` section is used to specify how detail the log should be,
the value should be within `trace`, `debug`, `info`, `warn`, `error`,
`critical`, and `off`.

The `trace` will display information to trace how program runs (most
detailed) and `critical` will only show information that may affect
system to crash (most brief).

Sample config:

``` shell
logging {
  directory = "/home/dora/log/discovery"
  file = "discovery.log"
  size = 1048576
  count = 20
  levels {
    DEFAULT = "info"
  }
}
```

bitcoind
--------

This section is described for version `0.17`. Since `bitcoind` provides
many detailed configuration, this second will mostly focus on settings
related to `bitmarkd`. Many resources can be found on internet for
information, the [official
site](https://bitcoin.org/en/developer-glossary) also provides some
explanation.

The `testnet` is used to denote `bitcoind` runs in test mode.

The `regtest` is used to denote `bitcoind` run in regression test mode.

The `dnsseed` is used to specify use DNS server to find other nodes to
connect.

The `dns` is used to allow DNS to be queried or not.

The `mintxfee` and `maxtxfee` are used to specify transaction fee.
Please refer to [wiki](https://en.bitcoin.it/wiki/Transaction_fees).

The `server` is used to enable a rpc server or not.

The `rest` is used to enable a rest http service or not.

The `[test]` means following attributes are only applied to `testnet`.
This section is newly added in `v0.17`.

The `port` is used to specify which port should peer connect with.

The `rpccport` is used to specify which port should rpc request connect
with. This value must be same as configuration in `discovery`.

The `rpcallowip` is used to specify with IP address is able to connect.
If multiple IPs should be allows, use multiple `rpcallowip` to denote.
It can also use dns mask to configure. Both network protocol of
ipv4/ipv6 are supported.

The zero MQ related fields (`zmqpubhashblock`, `zmqpubhashtx`,
`zmqpubrawblock`, `zmqpubrawtx`) are used to denote which IP and port
are used for zero MQ connection. These values must be same as
configuration in `discovery`.

`rpcuser` and `rpcpassword` are used for rpc authentication.

Sample config:

``` shell
# testnet
regtest = 0
testnet = 1

dnsseed = 1
dns = 1
upnp = 0

# fee settings
mintxfee = 0.00001
maxtxfee = 0.001
#paytxfee = 0.00001

# disable transaction index
txindex = 0
reindex = 0
prune = 1000

# run an rpc server
server = 1

# accept incoming peer connections
listen = 1

# enable the rest service
rest = 1

[test]
# peer port
port = 18333

# RPC configuration
rpcthreads = 5
rpcport = 18332
#rpcssl = 1
rpcallowip = 127.0.0.1/0
#rpcallowip=10.1.1.34/255.255.255.0
#rpcallowip=1.2.3.4/24
#rpcallowip=2001:db8:85a3:0:0:8a2e:370:7334/96
# test.rpcallowip = [::1]

# ZMQ configuration
zmqpubhashblock = tcp://127.0.0.1:18009
zmqpubhashtx = tcp://127.0.0.1:18009
zmqpubrawblock = tcp://127.0.0.1:18009
zmqpubrawtx = tcp://127.0.0.1:18009

# authentication
rpcuser = test
rpcpassword = testuser
```

litecoind
---------

Most of `litecoind` are similar to `bitcoind`. Many information can be
found on internet, so it will not be described further.

Be aware that `rpcport`, `zmqpubhashblock`, `zmqpubhashtx`,
`zmqpubrawblock`, `zmqpubrawtx` should also be same in `discovery`
configuration.

Sample config:

``` shell
# litecoin.conf for: coins.test.bitmark.com

# testnet
regtest = 0
testnet = 1
dnsseed = 1
dns = 1
upnp = 0

# fee settings
mintxfee = 0.00001
maxtxfee = 0.002
#paytxfee = 0.00001

# disable transaction index
txindex = 0
reindex = 0
prune = 1000


# run an rpc server
server = 1

# accept incoming peer connections
listen = 1

# enable the rest service
rest = 1


# peer port
port = 19335

# peer connections

# RPC configuration
rpcthreads = 5
rpcport = 19332
#rpcssl = 1
rpcallowip = 127.0.0.1/0
rpcallowip = [::1]

# ZMQ configuration
zmqpubhashblock = tcp://127.0.0.1:19009
zmqpubhashtx = tcp://127.0.0.1:19009
zmqpubrawblock = tcp://127.0.0.1:19009
zmqpubrawtx = tcp://127.0.0.1:19009

# authentication
rpcuser = test
rpcpassword = testuser
```