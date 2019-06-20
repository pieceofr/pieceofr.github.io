---
title: "The bitmark blockchain overview"
author_profile: true
excerpt: "A guild line for developer to add an attribute to the chain"
---

## Components
- `bitmarkd`: the main program for verifying and recording transactions in the Bitmark blockchain
- `recorderd`: an auxiliary application for computing the Bitmark proof-of-work algorithm that allows nodes to compete to win blocks on the Bitmark blockchain
- `discovery`: monitor cryptocurrencyies blockchains and publish possible payments txs to bitmarkd
- `bitcoind`
- `litecoind`
- `updaterd` (private)

## Networking

The communication of the above components is facilitated by [ZEROMQ](http://zeromq.org). Most connections are protected by the [CURVE mechanism](http://api.zeromq.org/4-0:zmq-curve) and require keypairs for authentication and encryption.

### Patterns used in bitmarkd

1. REQ-REP socket pair

    !["REQ-REP socket pair"](https://github.com/imatix/zguide/raw/master/images/fig2.png "REQ-REP socket pair")

2. PUB-SUB socket pair

    !["PUB-SUB socket pair"](https://github.com/imatix/zguide/raw/master/images/fig4.png)

3. PUSH-PULL socket pair

    !["PUSH-PULL socket pair"](https://github.com/imatix/zguide/raw/master/images/fig6.png)

| Role   | Action  | Socket Type     |
| ------ | ------- | --------------- |
| client | connect | REQ / SUB / PULL
| server | bind    | REP / PUB / PUSH

### bitmarkd & bitmarkd (P2P)


```shell
bitmarkd gen-peer-identity
```

Every running bitmarkd process acts as server and client at the same time.

- incoming connection ([REP](https://github.com/bitmark-inc/bitmarkd/blob/master/peer/listener.go))
- outgoing connection ([REQ](https://github.com/bitmark-inc/bitmarkd/tree/master/peer/upstream))

#### Network Discovery

> DNS seed: A DNS server which returns IP addresses of full nodes on the network to assist in peer discovery.

```shell
host -t txt nodes.live.bitmark.com 
``` 

#### Connection Dertermination

Each node is identified by its peer public key. Every node maintains its peers in AVL tree [^1] (including the node itself).

For example, the initial peer tree right after getting peers from the DNS seed:


![initial peer tree](https://i.imgur.com/djyTYCI.png)


![initial peer ring](https://i.imgur.com/X8okorU.png)


The peer choosing mechanism aims at providing evenly distribueted connections to share the load.

When receiving a new peer registration from the network, the node will add the new peer into the tree and re-determine the connections.

`peers-local.json`

### bitmarkd & recorderd


```shell
bitmarkd gen-proof-identity
recorderd generate-identity
```

- recorderd ([REQ](https://github.com/bitmark-inc/bitmarkd/blob/master/command/recorderd/submitter.go)) & bitmarkd ([REP](https://github.com/bitmark-inc/bitmarkd/blob/master/proof/submission.go)): for recorderd to submit mined blocks

- bitmarkd ([PUB](https://github.com/bitmark-inc/bitmarkd/blob/master/proof/publisher.go)) & recorderd ([SUB](https://github.com/bitmark-inc/bitmarkd/blob/master/command/recorderd/subscriber.go)): for recorderd to get potential blocks


### bitmarkd & discovery / bitcoind and liteoind

#### discovery

- discovery ([REP](https://github.com/bitmark-inc/discovery/blob/master/main.go#L119)) & bitmarkd ([REQ](https://github.com/bitmark-inc/bitmarkd/blob/master/payment/background.go#L180)): when bitmarkd just starts, it will query the latest possible txs for recovery

- discovery ([PUB](https://github.com/bitmark-inc/discovery/blob/master/bitcoin.go#L128)) & bitmarkd ([SUB](https://github.com/bitmark-inc/discovery/blob/master/command/payment-discovery/main.go#L24)): subscribe to possible payment txs (with correct OP_RETURN embeded) ([example](https://live.blockcypher.com/ltc/tx/b0675a796b457c400451f2a8ef7fdff67270eacfe92cf0da252ec68102544741/))


#### bitcoind and liteoind

- REST API

### bitmarkd & updaterd

```shell
bitmarkd gen-peer-identity // could reuse the peer keypair
updaterd generate-identity
```

- updaterd ([SUB](https://github.com/bitmark-inc/go-programs/blob/master/updaterd/peer/subscriber.go)) & bitmarkd ([PUB](https://github.com/bitmark-inc/bitmarkd/tree/master/publish))

[^1]: [The AVL tree visualization tool](https://www.cs.usfca.edu/~galles/visualization/AVLtree.html)

## Internal Communication

### message bus

![](https://i.imgur.com/kUjEh2A.png)

## Background process

- Each process implements the [backgroun process interface](https://github.com/bitmark-inc/bitmarkd/tree/master/background) and usually is started in `setup.go` under a package. For example, we have [two processes](https://github.com/bitmark-inc/bitmarkd/blob/master/peer/setup.go#L122) to handle incoming and outgoing connections.

## Storage

### Transicent (in-memory)

For pending and verified transaction. Before shutting down, these transactions will be written to a file `reservoir-local.cache` for reloading next time.

See more details in these packages:
- [asset](https://github.com/bitmark-inc/bitmarkd/tree/master/asset)
- [reservoir](https://github.com/bitmark-inc/bitmarkd/tree/master/reservoir)




### Persistent (leveldb)

1. block DB `testing-blocks.leveldb/`

![](https://i.imgur.com/t9oI7yD.png)


2. index DB `testing-index.leveldb/`

This can be rebuilt from the block db, mainly for efficient client query.

![](https://i.imgur.com/iP1ee4x.png)

See more details in these packages:
- [storage](https://github.com/bitmark-inc/bitmarkd/tree/master/storage)

## Transaction Records

https://github.com/bitmark-inc/bitmarkd/tree/master/transactionrecord