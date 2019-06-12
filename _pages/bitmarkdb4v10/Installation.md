---
title: "Install Full Node"
layout: single
permalink: /b4v10/Installation
sidebar:
  nav: "overview"
toc: true
---

Bitmark provide two ways to install full node. First, we provide a easy installation by using docker container. The container contains a bitmarkd, recorderd and bitmarkd utilities. There is a web api service inside the container. API can be used to control services. Another way to install full node is by installing each services. This gives user flexibility to configure full node.

## Prerequist

bitmarkd use directly IP connection to communicate with peers. To install a full node, the device needs a publick IP. Public IP can be a static or dynamic one. If public IP is changed, user needs to restart the bitmarkd.

## Install bitmark-node container

We have a detail installation and setup description in the [bitmark docker hub page](https://cloud.docker.com/u/bitmark/repository/docker/bitmark/bitmark-node) 

## Install full node services

In order to run full node, you need to run 3 services. If you needs to do payment and operation on create asset, transfer asset and etc. you need to install utilities. 

### Install bitmarkd
Here is our [bitmarkd project repo](https://github.com/bitmark-inc/bitmarkd). Readme will give you the detail installation information.

Master branch contains latest development code which may not test on our livenet or testnet. You should checkout specific version of release by using tag. You can use git tag --list to list out all release versions.

In general, you can checkout out the latest version of code and build it. To get correct version of livenet and testnet, user can build bitmark-info utilities and acquire information to a livenet or testnet node.

### Install recorderd

recoderd is also in the [bitmarkd project repo](https://github.com/bitmark-inc/bitmarkd). After installing bitmarkd, user can install recorderd in the bitmarkd/command/recorderd directory. 

### Discovery service

There are ways to install payment service. For a beginner using discovery proxy is recommanded.

+ Use discovery proxy

    User can turn on discovery proxy buy set the "use_discovery = true" in the bitmarkd.conf file.

+ Use internal discovery and install litecoind and bitcoind locally
    To run this mode, user needs to set the "use_discovery = false" and set up the the connection to litecoind and bitcoind. Find how to install bitcoind and litecoind in their official website.
    + [bitcoin](https://bitcoin.org/)
    + [litecoin](https://litecoin.org/)
    
+ Use local discovery proxy, litecoin and bitcoin services.
    User can install [discovery proxy](https://github.com/bitmark-inc/discovery) in your local device. In this configuration, user needs to install bitcoin and litecoin services . discovery proxy needs to connected to these two services. Find how to install bitcoind and litecoind in their official website.
    
    + [bitcoin](https://bitcoin.org/)
    + [litecoin](https://litecoin.org/)