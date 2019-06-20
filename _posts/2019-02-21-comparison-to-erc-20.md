---
title: "Comparison To ERC 20"
---

### Bitmark Issue Throughput

```
1 share requires 2 blocks to complete, first block for asset & issue record, second block for share record

max shares per hour
= num of blocks generated in one hour * maximum num of issues per block
= (60 min / 3 min) * (10000 txs per block * 1/3 shares per block)
= 66,666
```

* Ethereum Issue Throughput

```
"The average blocktime is about 15s.
The current gas limit is about 8M.
 Contract use 1.6M.

maximum # of contract per block
= 8M / 1.6M
= 5

maximum contract per block
= (3600 / 15) * 5
= 1200
```

### Bitmark Transfer (Grant & Swap) Throughput

```
grant needs a transfer and counter-sign, to it takes 2 transactions for transfering tokens

max pending transfers per hour
= num of blocks generated in one hour * maximum num of transfers per block
= (60 min / 3 min) * (10000 tx * 1/2)
= 100,000
 max confirmed transfers per hour
= min(max pending transfers per hour, throughput on BTC and LTC)
= min(100,000, (25,200 + 96,000))
= 100,000
```

### Ethereum Transfer Throughput

```
"The average blocktime is about 15s.
The current gas limit is about 8M.
Non-exist account transfer costs 52k, existing account transfer costs 37k, on average takes 45k gas.

maximum # of transfer per block
= 8M / 45k
= 177

maximum transfer per block
= (3600 / 15) * 177
= 42480
```

### Ethereum Swap Throughput

```
Approval uses 40k gas, transferFrom uses 55k gas

2 approval and 2 transferFrom are used:
(40k + 55k) * 2 = 195k gas

maximum # of swap per block
= 8M / 195k
= 41

maximum transfer per block
= (3600 / 15) * 41
= 9840
```

### Bitmark Issue Storage (bytes)

```
level db storage:
  - Asset: A + asset ID + block number + packed data = 1 + 64 + 8 + 162 = 235
  - Issue: T + tx ID + block number + packed data = 1 + 32 + 8 + 166 = 207
  - Share: F + tx ID + value + tx ID = 1 + 32 + 8 + 32 = 73

blockchain storage: 
  - asset: 162 bytes
  - issue: 166 bytes
  - share: 234 bytes
```

![Storage](Storage.png)

### Ethereum Issue Storage (bytes)

```
level db storage:
  - contract setup: 377 bytes
  - transaction: 5418 bytes

blockchain storage: 604 bytes
```

### Bitmark Transfer Storage (bytes)

```
level db storage:
  - Share: 
        F + tx ID + value + tx ID = 1 + 32 + 8 + 32 = 73
        Q + owner + tx ID + value = 1 + 33 + 32 + 8 = 74
  - Grant:
        new: Q + owner + tx ID + value = 1 + 33 + 32 + 8 = 74
        existing: 0

blockchain storage:
  - grant: 270 bytes
```

### Ethereum Transfer Storage (bytes)

```
level db storage: 139 bytes

blockchain storage: 603 bytes
```

### Bitmark swap storage

```
level db storage:
  - worst case is creating 2 new balance record
     2 * (Q + owner + tx ID + value ) = 2 * (1 + 33 + 32 + 8) = 148

blockchain storage: 270 bytes
```
### Bitmark Issue Cost

```
1 issue: 0.002 LTC (0.001 + 0.001)
1 share: 0.002 LTC (0.001 + 0.001)
```

### Bitmark Transfer

```
1 swap: 0.003 LTC
```

### Ethereum Transfer

```
0.0014 ETH (Approve cost + Transfer cost = 45397 + 29169 wei)
```

### Bitmark Swap

```
1 swap: 0.003 LTC
```

### Eth Swap

```
0.004 ETH
```