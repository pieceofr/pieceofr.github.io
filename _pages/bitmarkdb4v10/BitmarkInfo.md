---
title: "bitmark-info"
layout: single
permalink: /b4v10/BitmarkInfo
sidebar:
  nav: "utilities"
toc: true
---

User can use bitmark-info to get a peer node basic information locally or remotely. Query result returns information of network, block mode, rpc connection count, peer count, number of pending transcation and verified transcation, difficulty level, bitmarkd version number, peer node uptime and peer node publick key.

### Installation Steps

+ go to bitmarkd project
+ change directory to commands/bitmark-info
+ go install bitmark-info

### bitmark-info Usage

```
usage: bitmark-info [--help] host:port
```

### Example 

+  Get our bitmark test node-a1 info by

```
bitmark-info node-a1.test.bitmark.com:2130
```

+ Better format by using jq

```
bitmark-info node-a1.test.bitmark.com:2130 | jq
```

+ Query Result

```
{
  "host": "tcp://node-a1.test.bitmark.com:2130",
  "info": {
    "chain": "testing",
    "mode": "Normal",
    "blocks": 24124,
    "rpcs": 2,
    "peers": 19,
    "transactionCounters": {
      "pending": 44,
      "verified": 1
    },
    "difficulty": 1,
    "version": "0.10.7",
    "uptime": "174h18m46.855810566s",
    "publicKey": "57e97fb1e3b7bac43e170f79a77ea0f53e4b528a145d5a0d4693af8dba60181a"
  }
}

```