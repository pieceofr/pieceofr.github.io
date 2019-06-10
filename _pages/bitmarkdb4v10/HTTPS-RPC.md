
This document is based on bitmarkd **v8.1**.

- [POST /bitmarkd/rpc](#POST-bitmarkdrpc)
- [GET /bitmarkd/details](#GET-bitmarkddetails)
- [GET /bitmarkd/peers](#GET-bitmarkdpeers)
- [GET /bitmarkd/connections](#GET-bitmarkdconnections)

Since the the GET APIs return sensitive information about the running node, you need to explictly specify the allowed hosts in the bitmarkd configuration, otherwise the APIs are not accessible.

## POST /bitmarkd/rpc

This is an HTTP interface for performing supported JSON RPC calls in bitmarkd. For the usage of JSON RPC calls, please refer to [Bitmarkd RPC](https://github.com/bitmark-inc/bitmarkd/wiki/RPC).

### Parameters

none

### Body

The JSON RPC request object

### Returns

`Object` - The JSON RPC Reply

### Example

```shell
curl -k -X POST \
  https://127.0.0.1:2131/bitmarkd/rpc \
  -d '{
	"id": 1,
	"method": "Transaction.Status",
	"params":[
		{"txId": "2f2fe445e3c061fa9c5d42637431e988be83a44a90b677affd9ba2c140d0ff6e"}
	]
  }'
```

```json
{
    "id": 1,
    "result": {
        "status": "Confirmed"
    },
    "error": null
}
```

## GET /bitmarkd/details

Returns bitmarkd runtime details

### Parameters

none

### Returns

`Object` - The runtime details

- `chain`: possible values are `bitmark`, `testing` and `local`, [see the corresponding definition](https://github.com/bitmark-inc/bitmarkd/wiki/Module-overview#chain)
- `mode`: possible values are `Normal`, `Resynchronise` and `Stopped`, [see the corresponding definition](https://github.com/bitmark-inc/bitmarkd/wiki/Module-overview#mode) 
- `blocks`: the local and peer block height (the max block height among the peers)
- `rpcs`: number of rpc connections
- `peers`: number of incoming and outgoing connections
- `transactionCounters`:
    - pending: number of txs which are not paid yet
    - verified: number of txs which are already paid and ready to be added to the blockchain
- `difficulty`: [see the definition](https://github.com/bitmark-inc/bitmarkd/wiki/Module-overview#difficulty)
- `version`: the bitmarkd version
- `uptime`: the uptime of the node
- `publicKey`: the peer public key of the node

### Example

```shell
curl -k https://127.0.0.1:2131/bitmarkd/details
```

```javascript
{
  "chain": "testing", // chain = bitmark|testing|local
  "mode": "Resynchronise", // mode = Normal|Resynchronise|Stopped
  "blocks": {
    "local": 2501,
    "remote": 9802
  },
  "rpcs": 1,
  "peers": {
    "incoming": 1,
    "outgoing": 4
  },
  "transactionCounters": {
    "pending": 0,
    "verified": 0
  },
  "difficulty": 1,
  "version": "8.1",
  "uptime": "32m27.724531545s",
  "publicKey": "53352ac2943dd2426eb8c2a36e3c71f2061c67e0f62bace2df585b92e4c71a38"
}
```

## GET /bitmarkd/peers

Lists all peers and their public key in the network (sorted by peer public key).

### Parameters
 
| Parameter | Description | Format |
| --------- | ----------- | ----- |
| public_key | Returns peers with public key greater than the specified key | 32 byte hex-encoded peer public key string |
| count | The number of peers to return |default = 10, min = 1, max = 100|

### Returns

`Array` - Array of available peers.

| Attribute | Description |
| --------- | ----------- |
| publicKey | The peer public key of the peer node |
| listeners | The list of available connecting endpoints |
| timestamp | The time when the peer is added to the peer list |


### Example

```shell
curl -k https://127.0.0.1:2131/bitmarkd/peers
```

```json
[
  {
    "publicKey": "53352ac2943dd2426eb8c2a36e3c71f2061c67e0f62bace2df585b92e4c71a38",
    "listeners": [
      "118.163.120.175:2136"
    ],
    "timestamp": "2018-10-11T14:29:28.976409+08:00"
  },
  {
    "publicKey": "57e97fb1e3b7bac43e170f79a77ea0f53e4b528a145d5a0d4693af8dba60181a",
    "listeners": [
      "13.112.35.158:2136",
      "[2406:da14:da1:161c:75d2:28f7:7260:8cd1]:2136"
    ],
    "timestamp": "2018-10-11T14:29:28.975009+08:00"
  }
]
```

## GET /bitmarkd/connections

Lists all outgoing peer connections

### Parameters

none

### Returns

`Array` - Array of outgoing peer connections.

| Attribute | Description |
| --------- | ----------- |
| address | The IP address of the connected node |
| server | The peer public key of the connected node |

### Example

```shell
curl -k https://127.0.0.1:2131/bitmarkd/connections
```

```json
{
  "connectedTo": [
    {
      "address": "tcp://[2406:da14:da1:161c:75d2:28f7:7260:8cd1]:2136",
      "server": "57e97fb1e3b7bac43e170f79a77ea0f53e4b528a145d5a0d4693af8dba60181a"
    },
    {
      "address": "tcp://139.59.224.237:2136",
      "server": "8c4b3f6075c68b89c3ef3e2f2a1df8fdf728189d58996b45bdf418f71aef2441"
    },
    {
      "address": "tcp://[2400:6180:0:d1::61b:b001]:2136",
      "server": "d41e9fa3b1ad6fc3bb1239b5c0077e0c971b21e81168dd6212b83afdc7b3c710"
    }
  ]
}
```