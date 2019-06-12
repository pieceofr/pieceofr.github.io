---
title: "Bitmarkd Configuration"
layout: single
permalink: /b4v10/BitmarkdConf
sidebar:
  nav: "overview"
toc: true
---

## Config file format

The configuration language is Lua. The basic data structure in Lua is [table](https://www.lua.org/pil/2.5.html).

Use table to represent an array:

```lua
fibonacci = {1, 1, 2, 3, 5, 8, 13, 21}
```

Use table to represent a map:

```lua
family = {
    sister = {
        name = "Lisa"   
    },
    father = {
        name = "nick"   
    },
    mother = {
        name = "Elsa"
    }
}
```

### Sample

See the [config example](https://github.com/bitmark-inc/bitmarkd/blob/master/command/bitmarkd/bitmarkd.conf.sample).

#### Helper functions

In the sample, two extra functions are provided:

- `read_file(name)`: Read the file named by `name` under the specified data directory `M.data_directory` and return the content. This is usally used for reading key files.

- `announce_self(port)`: Let the node announce itself `ip:port` to the network. The ip should be provided using environment variables either PUBLIC_IPV4 or PUBLIC_IPV6, or both depends on the available public IP addresses of the node.

## Base Section

`M.data_directory`: All certificates, logs and LevelDB files should be relative to this directory.

`M.pidfile`: The location in which *bitmarkd* keeps its pid file.

`M.chain`: Select the chain of the network for peer connections. Possible values are:

- "bitmark": the main network
- "testing": the global testing network which mostly mimics the main network
- "local": a local testing network

`M.nodes` (default = "chain"): Specify the booting nodes. *bitmarkd* will lookup the DNS TXT records for available peers. Possible values are:

- "none": No booting nodes.
- "chain": Connect to a predefined set of nodes, which depends on `M.chain`

| `M.chain` | See available peer nodes for initializetion |
| - | - |
| bitmark | `host -t txt nodes.bitmark.com` |
| testing | `host -t txt nodes.test.bitmark.com` |

- other: The DNS server that provides the DNS TXT records of available booting nodes. Use `bitmarkd dns-txt` to get the record of the node.

`M.reservoir_file` (optional): The backup file for transactions in mempool.

`M.peer_file` (optional): To backup all peers into a file, which can be used for restore peer connections quicky if the *bitmarkd* if restarted.

## `M.client_rpc` Section

This section defines the functionality of [JSON RPC](https://github.com/bitmark-inc/bitmarkd/wiki/RPC).

`maximum_connections` (default = 100): Maximum connections to RPC clients.

`bandwidth` (default = 25 * 1000000): Set the rate limiting (MPS).

`listen`: Accepted incoming connections from RPC clients.

`announce`: Announce the RPC service of the node to the network. If not set, the service is not accessible to other peers (won't be shown in the response for RPC [Node.Info](https://github.com/bitmark-inc/bitmarkd/wiki/RPC#nodelist)).

`certificate` and `private_key`: Required certificate and key for setting up secure TCP connections.

## `M.https_rpc` Section

This section defines the functionality of [HTTPS RPC](https://github.com/bitmark-inc/bitmarkd/wiki/HTTPS-RPC).

The configuration is similar to [client_rpc](#Mclient_rpc-Section). The only difference is that some APIs return sensitive information about the running node, you need to explictly specify the allowed hosts in the configuration, otherwise the APIs are not accessible.

See [the current available API](https://github.com/bitmark-inc/bitmarkd/wiki/HTTPS-RPC).

`allow`: Accepted incoming connections for respective APIs

- `details`: for GET /bitmarkd/details
- `connections`: for GET /bitmarkd/details
- `peers`: for GET /bitmarkd/peers


## `M.peering` Section

This section defines the P2P behavior of *bitmarkd*.

`dynamic_connections` (default = true): Set to false to prevent additional connections. If dynamic connections are disabled, static connections should be specified in `connect`, otherwise *bitmarkd* won't have enough peers for synchronization.

`prefer_ipv6` (default = true): Set to false to only use IPv4 for outgoing connections.

`listen`: Accepted incoming connections for P2P communication.

`announce`: Announce the node to the network. If not set, the node will can still get access to the network, but not accessible to other nodes.

`public_key` and `private_key`: Required keys to construct secure connections with peers.

`connect` (optional): Dedicated static connections to a set of nodes.

## `M.publishing` Section (optional)

This section defines how *bitmarkd* publishes blocks and transactions.

`broadcast`: Accepted incoming connections to subscribe blocks and transactions.

`public_key` and `private_key`: Required keys to construct secure connections with clients.
    
## `M.proofing` Section (optional)

This section defines how *bitmarkd* works with *recorderd* and receive monetary compensation. If you don't plan to participate in mining blocks, this section can be ignored.

![connections between bitmarkd and recorderd](proof.png)

`public_key` and `private_key`: Required keys to construct secure connections with *recorderd*.

`signing_key`: Used to sign the published potential block.

`payment_address`: The cryptocurrency addresses to receive payments for mined blocks.

`publish`: Accepted incoming connections to subscribe potential blocks.

`submit`:  Accepted incoming connections to submit mined blocks.

## `M.payment` Section

This section defines how *bitmarkd* receives the payment transactions for Bitmark issues and transfers.

There are two ways to configure the payment section.

![possible payment configuration](payment.png)

### Connect to the discovery proxy (1)

[The discovery proxy](https://github.com/bitmark-inc/discovery) monitors bitcoin and litecoin transactions and publish the possible transactions which are used to pay Bitmark issue and transfer transactions.

Configuration Example:

```lua
M.payment = {
    use_discovery = true,

    discovery = {
        sub_endpoint = "127.0.0.1:5566", 
        req_endpoint = "127.0.0.1:5567"
    }
}
```

`use_discovery`: Set to true to get payment transactions directly from *discovery*.

`discovery`: Connect *bitmarkd* to *discovery*.

- `sub_endpoint`: Set to the [pub_endpoint](https://github.com/bitmark-inc/discovery/blob/master/discovery.conf.sample#L1) of *discovery*.

- `req_endpoint`: Set to the [rep_endpoint](https://github.com/bitmark-inc/discovery/blob/master/discovery.conf.sample#L3) of *discovery*.

### Connect to bitcoind and litecoind (2)

Running bitcoin and litecoin nodes are required. Make sure the REST interface are enabled.

```shell
bitcoind -rest -rpcport=<port>
litecoind -rest -rpcport=<port>
```

Configuration Example:

```lua
M.payment = {
    use_discovery = false,

    bitcoin = {
        url = "http://<bitcoind_host>:<rpcport>/rest"
    },
    litecoin = {
        url = "http://<litecoind_host>:<rpcport>/rest"
    }
}
```

## `M.logging` Section

The default configuration is as follows.

```lua
M.logging = {
    size = 1048576,
    count = 10,
  
    console = false,
  
    levels = {
        DEFAULT = "critical"
    }
}
```

`size`: Rotate the log file if it exceeds this size (bytes).

`count`: Maximal count of retained log files. When this limit is exceeded, older rolls are removed.

`console`: Set to true for writing logs to the console.

`levels`: Define the log level of each module. If a module is not configured, the DEFAULT level will be used for logging.

The format of *bitmarkd* logs is `DATE TIME [LOG_LEVEL] MODULE: MESSAGE`. 

For example,

```
2018-11-15 10:55:13 [INFO] publisher: verified pool is empty
2018-11-15 10:55:27 [INFO] connector: current state: Sampling
2018-11-15 10:55:27 [INFO] connector: connections: 11
2018-11-15 10:55:27 [INFO] connector: height  remote: 11149  local: 11149
2018-11-15 10:55:27 [INFO] upstream@2: height: 11149
2018-11-15 10:55:27 [INFO] upstream@6: height: 11149
```

The configurable modules are `announcer`, `aux`, `bitcoin`, `block`, `blockstore`, `broadcaster`, `checker`, `connector`, `discoverer`, `listener`, `litecoin`, `main`, `memory`, `publisher`, `ring`, `rpc`, `submission`, `upstream@N` (N is in the range [1..m] if there are m peers). 
