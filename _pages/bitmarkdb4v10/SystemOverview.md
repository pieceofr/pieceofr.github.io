---
title: "Bitmark Full Node Overview"
layout: single
permalink: /b4v10/overview
sidebar:
  nav: "overviewIndex"
toc: true
---

## The Full Node
![bitmark full node](https://i.imgur.com/TigGn2e.jpg)

Bitmark Full Node consist with **bitmarkd**, **recorderd** and **payment system**. bitmarkd is the core of bitmark blockchain. Its major task is to **generate blockchain** and **synchronize** with other bitmark node. Recorderd is like **miner** in bitcoin. It caculates hash value to confirm the block and delivered confirmed block to bitmarkd. Like bitcoin miner, it competes with other recorderd to win a block. Bitmark blockchain does not have our own crypto-currency embeded. **We use other crypto-currencies to pay for transaction fee**. We called the part to handle other crypto-currency payment as payment system. Currently we support **bitcoin** and **litecoin**.


## Digital Property Blockchain

bitmark blockchain is specifically designed for digital property. Before to have a deeper understanding of our technology, we suggest developers to understand the basic terms of digital property in bitmark blockchain.


+ **Asset Record**
In bitmark blockchain, asset is an temporary record of your properties record. If you have no further action on issuing it, it will be expired after a time interval.

+ **Issue Record**
Asset becomes Issue Record after it has issued. An Issue Record is recorded in blockchain. Issue a record is like transaction which need be confirmed by recorderd/miner. Therefore, it should be charged some fee for transaction. Bitmark cover this fee for users at the moment.

+ **Transfer Record**
Issue Record confirm the ownership of the creater. In the reality we can transfer the property to other people. In bitmark blockchain, a digital property can be transfered to other user too. The transfer of property will be recordered in Transfer Record of blockchain.

+ **Provenance**
In bitmark blockchain, every digital property is traceable. Provenance has history of a digital property. Providing an Transaction ID, bitmark can give user the Provenance of a digital property.

![](https://i.imgur.com/G6aFNSY.jpg)

## The Entities

### bitmarkd
bitmarkd is a peer or node of the bitmark blockchain. It can be a stand-alone service. Running bitmarkd alone, user can synchronize with other peers or nodes to get the latest blockchain. User can also use bitmark utilities to create issue and transfer record and get the blockchain information.

### discovery

discovery service periodically asks bitcoind node and litecoind node the information of payment info of bitmark users. If discovery service has comfirmed that a bitmark transcation has paid from bitcoind/litecoind, it informs bitmarkd that the user has paid. After bitmarkd has recieved a transaction has paid from discovery, it add the transaction to block and broadcast it to other bitmark nodes. Meanwhile, it send sthe block to the recorderd for mining.
bitmark full node support two kinds of discovery service - discovery proxy and discovery inside bitmarkd. When user uses proxy mode of discovery service, the bitmarkd connect to a bitmark discovery proxy in the cloud and wait for its notification for payment messages. If user uses bitmarkd internal discovery service, bitmarkd ask bitcoind/litecoind server the payment info. In order to do the asking, user may need to setup their own bitcoind and litecoin servers. In general, using discovery proxy is a easier setting-up.

![two configuration of discovery](https://i.imgur.com/Vfy7yW4.jpg)

### recorderd
recorderd 自 bitmarkd 接獲 Verified 的 Issue Record 或 Transfer Recorded. recorderd 負責計算hash value 來確認這些 block 是有效的. 當內含多個的 Records 的 block 在 recorderd 進行運算後, 這個 block 會被送回 Bitmarkd. Bitmarkd 會將這個 block 寫入blockchain裡.

### Utilities
#### bitmarkd-cli
#### bitmarkd-info
#### bitmarkd-dumpdb
#### bitmark-wallet

## Node Communication

## Source Tree

###### tags: `bitmarkd` `full node` `documentation` `overview` `english`