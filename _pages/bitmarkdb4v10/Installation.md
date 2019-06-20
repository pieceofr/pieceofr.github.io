---
title: "Install Full Node"
layout: single
permalink: /b4v10/Installation
sidebar:
  nav: "overview"
toc: true
---

Bitmark provides two ways to install full-node. We provide an **easy** installation by using **docker container**. The container contains a bitmarkd, recorderd and bitmarkd utilities. There is an API service inside the container to be used to control services.  Another way to install full-node is by **installing each service**.  An user can have more **flexibility** to configure full-node in this way.

## Prerequisite

Bitmarkd uses a direct IP connection to communicate with peers. In order to run a full-node, the **device needs a public IP** which can be a static or dynamic one. If public IP is changed, the user needs to restart the bitmarkd or to run the bitmark-node container again.

## Install bitmark-node container

Here is a detail installation and setup description in the [bitmark docker hub page](https://cloud.docker.com/u/bitmark/repository/docker/bitmark/bitmark-node) 

## Install full node services

In order to run full-node, a user has to install and set up three services. In addition, if the user wants to do operations like creating an asset, transferring an asset and etc., bitmark command utilities may require to be installed according to their needs.

### Install bitmarkd
Here is our [bitmarkd project repo](https://github.com/bitmark-inc/bitmarkd). Readme has detail installation steps.

Master branch contains the latest development code which may not test on our livenet or testnet yet.  User can checkout a specific version of release by a GitHub tag. You can use git tag --list to list out all released versions of bitmarkd.
In general, you can use the latest released version. To get a correct version of livenet and testnet, user can build bitmark-info, a command line utility, which can acquire information from a livenet or testnet node. 

### Install recorderd

Recoderd is in the [bitmarkd project repo](https://github.com/bitmark-inc/bitmarkd).  After installing bitmarkd, user can install recorderd in the bitmarkd/command/recorderd directory by ```go install command```.

### Install discovery

There are ways to install payment service. For a beginner, discovery proxy is recommended.

+ Use discovery proxy

    User can turn on discovery proxy by setting the "use_discovery = true" in the bitmarkd.conf file.

+ Use internal discovery and install litecoind and bitcoind locally
    To run this mode, a user needs to set the "use_discovery = false" and set up the connection to litecoind and bitcoind in payment section in bitmarkd.conf file. For detail information, check [bitmarkd detail configuration](/b4v10/BitmarkdConf) section.
    
+ Use local discovery proxy with litecoind, and bitcoind.
    User can install [discovery proxy](https://github.com/bitmark-inc/discovery) in your local device. In this configuration, the user set up their own discovery proxy. The user has to install bitcoind and litecoind and configure the discovery proxy to connect to both services.For detail information, check [discovery detail configuration](/b4v10/DiscoveryConf) section.

+ Find how to install bitcoind and litecoind on their official website.    
    + [bitcoin](https://bitcoin.org/)
    + [litecoin](https://litecoin.org/)
  
