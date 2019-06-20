---
title: "Comparison to ERC 721"
author_profile: true
---
Bitmark vs ERC 721
### Bitmark Issue

```
max issues per hour
= num of blocks generated in one hour * maximum num of issues per block
= (60 min / 3 min) * (10000 txs per block ÷ 2 txs/issue)
= 100,000
```

### Ethereum Issue
```
maximum num of issues per block
= (gas limit / gas used by transaction) * maximum number of issues in a tx
= (8000000 / 7816529) * 37
~ 37

num of issues per hour
= num of blocks generated in one hour * maximum num of issues per block
= (3600 sec / 15 sec) * 37
= 8,880
```

### Bitmark Transfer
```
max pending transfers per hour
= num of blocks generated in one hour * maximum num of transfers per block
= (60 min / 3 min) * 10000
= 200,000
 max confirmed transfers per hour
= min(max pending transfers per hour, throughput on BTC and LTC)
= min(200,000, (25,200 + 96,000))
= 121,200
```

### Ethereum Transfer
```
maximum num of tranfers per block
= gas limit / gas used by transaction
= 8000000 / 150000
~ 53 
num of transfers per hour
= num of blocks generated in one hour * maximum num of transfers per block
= (3600 sec / 15 sec) * 53
= 12,720
```

### Bitmark Storage (bytes)

8,602

![bitmarkd storage](Storage.png)

### Ethereum Storage (bytes)

47,195

### Bitmark Issue Cost
```
Miner fee for 1 issue: 0.002 LTC (0.001 + 0.001)
Miner fee for 100 issue: 0.101 LTC (0.001 * 100 + 0.001)
```

### Ethereum Issue Cost
```
the issue tx costs is 32 issues: 0.16954365 ETH
the issue tx costs of 100 issues: 0.50863 ETH (0.16954365 * 3)
```

###* Bitmark Transfer
```
the cost of a transfer = 0.003 LTC
```

### Ethereum Transfer
```
the transfer tx costs 0.003798875 ETH  
```