---
title: "bitmark-dumpdb"
layout: single
permalink: /b4v10/DumpDB
sidebar:
  nav: "utilities"
toc: true
---

bitmark-dumpdb is an utility to dump bitmark leveldb. 

### Usage

```usage: bitmark-dumpdb [--help] [--verbose] [--quiet] [--count=N] --file=FILE tag [--list] [key-prefix]```

### Examples

+ List types of data

  ``` bitmark-dumpdb --list ```

  ``` 
        B → Blocks
        H → BlockOwnerPayment
        I → BlockOwnerTxIndex
        A → Assets
        T → Transactions
        N → OwnerNextCount
        L → OwnerList
        D → OwnerTxIndex
        O → OwnerData
        F → Shares
        Q → ShareQuantity
        Z → TestData
  ```

+ Read Asset
  + "file" is path for leveldb file. bitmark in the end of file is a surfix of leveldb. In the case, the bitmark-dump looks for $HOME/.config/bitmarkd/data/bitmark-blocks.leveldb and $HOME/.config/bitmarkd/data/bitmark-index.leveldb
  + "count" is the number of items to show
  
  ```
    bitmark-dumpdb --verbose --count=1 --file=$HOME/.config/bitmarkd/data/bitmark A
  ```

  + Result

  ```
    0: Key: 000002352a3ab579781dc4df26c148b553b827fc53eebed9fe2db8196d6a6dad1fcbda9495dd2421d51f94c68d24152b561a511a923646fd310fb9fa8e3d602c
    0: Val: 0000000000008c67020b696f7337323334343835338201303137646165336636306633343231383264643062373464613237613631336537663963363239653631333936383266303637663962366330353835323834643530306535356236306162323830336661363664353131383234653137373235663038636164643063346138313738383638616463623730386235303862386231331d736f7572636500494f532070686f746f007479706500506963747572652111ba037089f8f6d1938d7a65a3e0a07f1878811f7e804676a9a4d5e4c90b65d6554069c03a0bf6f99553e75f854ec0c0ceda38f419b7130de69e5e8e1a8e02ceb80cd1979e7c12870bfd1f4180793230e3d40948c9dea6c6fc0851cb40735239d102

  ```