---
title: "Bitmark Full Node Overview"
layout: single
permalink: /b4v10/FullNodeOverview
sidebar:
  nav: "overview"
toc: true
---
## The Full-Node

![bitmark full-node](https://i.imgur.com/TigGn2e.jpg)

Bitmark full-node consists of services of **bitmarkd, recorderd, and payment system**. Bitmarkd is the core of bitmark blockchain. It **generates blockchain and synchronizes** with other bitmark nodes. Recorderd works like a **miner** in bitcoin. It calculates hash value received from bitmarkd and when it finds a possible solution, it sends the hash back to the bitmarkd. Like bitcoin miner, it competes with other recorderd to win a block. Bitmark blockchain does not have our own crypto-currency. We **use other crypto-currencies to pay for transaction fee**. We called the part to handle other crypto-currency payment as a payment system. Currently, we support **Bitcoin and Litecoin** payment.


## Digital Property Blockchain

Bitmark blockchain is specifically designed for digital property. We suggest developers understand the basic terms of digital property in bitmark blockchain.


+ **Asset Record**
In a bitmark blockchain, an asset is a temporary record of your properties record. If you have no further action on issuing it, it will be expired after a time interval.

+ **Issue Record**
An asset becomes an Issue Record after it has issued. An Issue Record is recorded in the blockchain. To become an issued record,  the asset will be confirmed by recorderd/miner; therefore, it will be charged a fee for the transaction. Currently, Bitmark Inc. covers the fee for the first asset issued.

+ **Transfer Record**
Issue Record confirms the ownership of the creator. In the real world, we can transfer our properties to other people. In the bitmark blockchain, digital properties can be also transferred to other users. The transfer of property will be recorded in Transfer Record of blockchain.

+ **Provenance**
In a bitmark blockchain, every digital property is traceable. Provenance has a complete history of digital property. By providing a transaction ID, bitmark can return the provenance of digital property.


![](https://i.imgur.com/G6aFNSY.jpg)

## The Entities in a Full-Node

### bitmarkd
bitmarkd is a peer or node of the bitmark blockchain. It can be a stand-alone service. Running bitmarkd alone, the user can synchronize with other peers or nodes to get the latest blockchain. User can also use bitmark utilities to create an issue and transfer record and get the blockchain information.

### discovery

Discovery service periodically asks bitcoind and litecoind payment information of bitmark users. If discovery service has confirmed that a bitmark transaction has paid from bitcoind or litecoind, it informs bitmarkd the transaction fee is paid. After bitmarkd has received a message from discovery, it adds the transaction to block and broadcast it to other bitmark nodes. Meanwhile, it sends the block to the recorderd for calculating the hash value. Bitmark full-node supports two kinds of discovery service - discovery proxy and discovery inside bitmarkd. When a user uses the proxy mode of discovery service, the bitmarkd connects to a bitmark discovery proxy in the cloud and waits for notification from discovery proxy for payment information. If a user uses bitmarkd internal discovery service, bitmarkd asks bitcoind/litecoind server for the payment information. In order to do the asking, the user may need to set up their own bitcoind and litecoind servers. In general, using a discovery proxy is an easier setting-up.

![two configuration of discovery](https://i.imgur.com/Vfy7yW4.jpg)

### recorderd

bitmarkd packs the verified transactions into a block and send the block to the recorderd. Recorderd calculates the block with nonce and returns the possible solution;  then forward the solution back to the bitmarkd. bitmarkd verified the solution if it is a valid solution, bitmarkd add the block to the chain and broadcast to other peers.
