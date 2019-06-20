---
title: "Setup a bitmarkd local cluster on Mac"
author_profile: true
---

## Prerequsite

- bitmarkd dependencies

    - go 1.12+
    - argon2
    - zeromq

    Follow the instructions of the **prerequsite** section in the [bitmarkd repo](https://github.com/bitmark-inc/bitmarkd).

- DNS server

    - dnsmasq

    ```shell
    brew install dnsmasq
    ```

## Installation

```shell
go get github.com/bitmark-inc/bitmarkd
go install -v github.com/bitmark-inc/bitmarkd/command/bitmarkd
go install -v github.com/bitmark-inc/bitmarkd/command/recorderd
go install -v github.com/bitmark-inc/bitmarkd/command/bitmark-cli
```

## Run a local cluster

1. Download the template configuration file.
    - [bitmarkd](https://gist.github.com/nodestory/2f9d32359d595c76559c66df666083d5)
    - [recorderd](https://gist.github.com/nodestory/fe6c3a08fc14958b0d9af0daf5f2fe64)
2. Download the [shell script](https://gist.github.com/nodestory/dfd4165033be9508e210b73211883379) for running the cluster.
3. Modify the [payment addresses](https://gist.github.com/nodestory/2f9d32359d595c76559c66df666083d5#file-bitmarkd-local-conf-lua-L155) to your testnet bitcoin and litecoin addresses. (optional: if you don't mind you won't earn the mining rewards on testnet)

### Initialize the cluster

```shell
BASE_DIR=<dir> NODE_COUNT=<cnt> sh run_cluster.bash init
```

You have to specify a base directory for the data generated by bitmarkd and recorderd, and also the number of bitmarkd processes you want (it takes at least 4 to establish a working cluster) in the environmental variables.

This command will restart the local DNS server, so it prompts for the password.

Assume your base directory is `/local_cluster` and ask for 4 running nodes, then you can find the following directories with required keys generated and stored in the respective directory:
```
/local_cluster/bitmarkd/1
/local_cluster/bitmarkd/2
/local_cluster/bitmarkd/3
/local_cluster/bitmarkd/4
/local_cluster/recorderd
```

The first node adopts [the default configuration](https://github.com/bitmark-inc/bitmarkd/blob/master/command/bitmarkd/bitmarkd.conf.sample), and the following nodes shifts their ports by incrementing 100. For example, the HTTPS rpc port of the second node will be 2231.

### Start the cluster

```shell
BASE_DIR=<dir> NODE_COUNT=<cnt> sh run_cluster.bash start
```

Use [HTTPS RPC details](https://github.com/bitmark-inc/bitmarkd/wiki/HTTPS-RPC#GET-bitmarkddetails) to check the health of the node.


### Stop the cluster

```shell
sh run_cluster.bash stop
```