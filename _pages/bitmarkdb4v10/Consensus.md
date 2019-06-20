---
title: "Bitmark Protocol Overview"
layout: single
permalink: /b4v10/Concensus
sidebar:
  nav: "protocol"
toc: true
---

![Under Construction](/assets/images/ico_under_construction_512.png)



## Four Modules of Consensus
Consensus is composed of four modules, they are,

+ **block** : give the structure of block.
+ **block digest** : give the way the blcok header is hashed by argon2 and block storage routine that executes the whole block consensus algorithm
+ **block record** : give the structure of the header
+ **difficulty** : give the structure of difficulty-variable and how difficulty is compared

## Block

+ In setup.go, it give the structure of block. 
The block data is consisted with 

```
type blockData struct {
	sync.RWMutex // to allow locking

	log *logger.L

	height            uint64             // this is the current block Height
	previousBlock     blockdigest.Digest // and its digest
	previousVersion   uint16             // plus its version
	previousTimestamp uint64             // plus its timestamp
	rebuild           bool               // set if all indexes are being rebuild
	blk               blockstore         // for sequencing block storage

	// for background
	background *background.T

	// set once during initialise
	initialised bool
}
```

+ In store.go, a StoreIncoming routine checks and stores block into database.
```func StoreIncoming(packedBlock []byte) error```


The first half of the incoming routine is pure validation. It very carefully validate the block header. If the difficulty is correct and the time stamp is correct, the previous block hash is all line-up and then it validates every transactions to make sure they are ok. 

The function checks that the transfer is linked to transfer and issues, double  signature has done correctly and every transaction is in the block validate. It then checks the first transaction is a base record or a foundation record and make sure we get the Bitcoin and Litecoin for payment.

If they are missing, the block is not usable anymore and not be able to use any issue and transfer in that block in the future transaction. Only then, does it go to the transactions second times and write each one into the database and update the transfer that would move the ownership from one user to the next. This is very sensitive part of the operation because any panic here would lead database corrupt.

## BlockRecord Module

In blockrecord module, 
+ header.go  : defines structure of header
    + constants for the offset in the header
    + header structure
    + a rountine that extracting data from the header and verify it to make sure it does not violate any of header constraints.

## Difficulty Module

Difficulty is expressed as an uin64 which actually presents a reciprocal of floating point number. The integer part affectively guess smaller and smaller and it is converted into a 256 bits big number.This corresponds to the same length of bit of argon2 hash so we can do a directly comparison of argon2 hash and difficulty and figure out if the hash is lower than the difficulty value. 

It also has a reciprocal version. Because it is hard to work with tiny tiny fraction, we take the reciprocal so that we can consider the difficulty of one, two, three and four as increasing values. The most of difficulty routine is avoiding to store whole 32 bytes difficulty value inside the header because it just wastes space when it get enough accuracy with 8 bytes. 

The original bitcoin only use 4 bytes that causes some kind of  rounding errors so we went to next one up, uint64 and 8 bytes value. uint64 value is more nature value of Go code and we are using the 64 bits machine now so using 32 bits value is less efficient. 

The difficulty module is to define what is the difficulty one is , how to get the reciprocal, how to convert float-point difficult, how to convert difficulty value to 8 bits representation  into block header and convert 8 bytes block header to the 256bits so we can compare . Provide CNP function to do the comparison. 

+ **filter submodule**
In filter submodule,  camm filter tries to estimate difficulty over the time period of hour to another difficulty to recover quickly if the amount of mining power change suddenly. We don't want to wait day for the recovery if the mining power change 30-40%. In bitcoin, this amount of change of mining power may take 2 weeks to recover. It also means if you suddenly have a lot of mining power, you could earn a lot of blocks in a short time. To stop it happening every time, when the block is loaded, it sends to the filter to try to estimate what the current difficulty should be.The difficulty is adjusted slowly over the period from hour to the day to act like feedback control of difficulty so we response more quickly in change of hashing power. 



###### tags: `bitmark` `bitmarkd` `documentation` `consensus`