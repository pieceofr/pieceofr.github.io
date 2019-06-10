
# Table of Contents

1.  [Bitmarkd Peer](#org936fae5)
    1.  [Initialization](#orgec097ce)
        1.  [RPC Response Listener](#org56d7964)
        2.  [RPC Request Connector](#org31f2a46)
        3.  [Start to process incoming/outgoing messages](#org9b9ab28)
    2.  [Format of message](#orgb494e51)
    3.  [Process Listener message](#orgff8074f)
        1.  [Server information ("I")](#orgb223f5c)
        2.  [Get block number ("N")](#org668fca4)
        3.  [Get packed block ("B")](#org1c83679)
        4.  [Get block hash ("H")](#org0cbef05)
        5.  [Register to another node ("R")](#org6ac295e)
        6.  [Default action](#orgc200c7d)
    4.  [Connector Peer Processing](#orgc437bbe)
        1.  [Conneting](#org8f4abb0)
        2.  [Hightest block](#org644afb4)
        3.  [Fork detected](#orgc864745)
        4.  [Fetch block](#org75dc618)
        5.  [Rebuild](#org105a8e6)
        6.  [Sampling](#org86743a5)


<a id="org936fae5"></a>

# Bitmarkd Peer

Following paragraphs are described based on `v8.2` commit `b013d5e`.


<a id="orgec097ce"></a>

## Initialization

Flow starts at `bitmarkd/main.go#278: peer.Initialise`.

peer package contains `peerData` with read/write lock (`sync.RWMutex`),
rpc listeners (`lstn`), rpc clients (`conn`), etc.

```go
    type peerData struct {
        sync.RWMutex // to allow locking

        log *logger.L // logger

        lstn listener  // for RPC responses
        conn connector // for RPC requests

        connectorClients []*upstream.Upstream

        publicKey []byte

        clientCount int
        blockHeight uint64

        // for background
        background *background.T

        // set once during initialise
        initialised bool
    }

    type listener struct {
        log         *logger.L
        chain       string
        version     string      // server version
        push        *zmq.Socket // signal send
        pull        *zmq.Socket // signal receive
        socket4     *zmq.Socket // IPv4 traffic
        socket6     *zmq.Socket // IPv6 traffic
        monitor4    *zmq.Socket // IPv4 socket monitor
        monitor6    *zmq.Socket // IPv6 socket monitor
        connections uint64      // total incoming connections
    }
```

In `peer/setup.go: Initialise`, it reads private/public key file,
registers to `announce` which will broadcast events (`setAnnounce`), initialize
objects to process request/response.


<a id="org56d7964"></a>

### RPC Response Listener

`listener` is an object includes logger, ipv4/ipv6 sockets,
receiver/sender, ipv4/ipv6 monitors. Detail data can be found as
below:

```go
    type listener struct {
        log         *logger.L
        chain       string
        version     string      // server version
        push        *zmq.Socket // signal send
        pull        *zmq.Socket // signal receive
        socket4     *zmq.Socket // IPv4 traffic
        socket6     *zmq.Socket // IPv6 traffic
        monitor4    *zmq.Socket // IPv4 socket monitor
        monitor6    *zmq.Socket // IPv6 socket monitor
        connections uint64      // total incoming connections
    }
```

`listener` is initialized at `peer/setup.go#109: globalData.lstn.initialise`.

Initialization process as below:

1.  creates logger

2.  creates sender (push) and receiver (pull) for each socket

    At `peer/listener.go#71: zmqutil.NewSignalPair(listenerSignal)`.

    Be aware that `listenerSignal =
         "inproc://bitmark-listener-signal"` is defined by zeroMQ, `inproc`
    can be referenced [here](http://api.zeromq.org/4-1:zmq-bind#toc2), it denotes local in-process
    (inter-thread) communication transport.

3.  allocates ipv4/ipv6 sockets

    At `peer/listener.go#77: zmqutil.NewBind`

4.  sets monitors of ipv4/ipv6 each

    At `peer/listener.go#84: zmqutil.NewMonitor`.


<a id="org31f2a46"></a>

### RPC Request Connector

`connector` is an object includes logger, client lists, connection
state, other node's largest block height, etc.

connector state is an integer value to represents finite state
machine state.

```go
    type connector struct {
        log *logger.L

        preferIPv6 bool

        staticClients []*upstream.Upstream

        dynamicClients list.List

        state connectorState

        theClient        *upstream.Upstream // client used for fetching blocks
        startBlockNumber uint64             // block number where local chain forks
        height           uint64             // block number on best node
        samples          int                // counter to detect missed block broadcast
    }

    type Upstream struct {
        sync.RWMutex
        log         *logger.L
        client      *zmqutil.Client
        registered  bool
        blockHeight uint64
        shutdown    chan<- struct{}
    }
```

It is initialized at `peer/connector.go#65: initialise`.

Initialization process as below:

1.  create logger

2.  parse ip address and port

    At `peer/connector.go#87: util.NewConnection`

3.  create upstream client

    Inside `peer/connector.go#107: upstream.New` creates another go
    routine for function `upstreamRunner` which will wait for incoming messages.

4.  connect to server

    At `peer/connector.go#117: client.Connect`


<a id="org9b9ab28"></a>

### Start to process incoming/outgoing messages

It is started by go routine at `peer/setup.go#128:
    background.Start`. With in this function, it invokes each object's
method of `Run`.

```go
    go func(p Process, shutdown <-chan struct{}, finished chan<- struct{}) {
                // pass the shutdown to the Run loop for shutdown signalling
                p.Run(args, shutdown)
                // flag for the stop routine to wait for shutdown
                close(finished)
            }(p, shutdown, finished)
```

1.  Listener

    `listener` `Run` method is defined at `peer/listner.go#101: Run`, for different
    incoming message/event invokes different function:

```go
        for {
            sockets, _ := poller.Poll(-1)
            for _, socket := range sockets {
                switch s := socket.Socket; s {
                case lstn.socket4:
                    lstn.process(lstn.socket4)
                case lstn.socket6:
                    lstn.process(lstn.socket6)
                case lstn.pull:
                    s.RecvMessageBytes(0)
                    break loop
                case lstn.monitor4:
                    lstn.handleEvent(lstn.monitor4)
                case lstn.monitor6:
                    lstn.handleEvent(lstn.monitor6)
                }
            }
        }
```

2.  Connector

    `connector` `Run` method is defined at `peer/connector.go#182: Run`, for
    different outgoing message invokes different function:

```go
        for {
            // wait for shutdown
            log.Debug("waiting…")

            select {
            case <-shutdown:
                break loop
            case item := <-queue:
                c, _ := util.PackedConnection(item.Parameters[1]).Unpack()
                conn.log.Debugf("received control: %s  public key: %x  connect: %x %q", item.Command, item.Parameters[0], item.Parameters[1], c)
                //connectToUpstream(conn.log, conn.clients, conn.dynamicStart, item.Command, item.Parameters[0], item.Parameters[1])
                conn.connectUpstream(item.Command, item.Parameters[0], item.Parameters[1])

            case <-time.After(cycleInterval):
                conn.process()
            }
        }
```


<a id="orgb494e51"></a>

## Format of message

Typicall command will be as follow, parameters are append at last
of message with each 8 bytes long

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Command</th>
<th scope="col" class="org-left">Chain Mode</th>
<th scope="col" class="org-left">Parameters</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left">1 byte</td>
<td class="org-left">8 bytes</td>
<td class="org-left">8 bytes each</td>
</tr>
</tbody>
</table>

e.g. When register to another node, the command will be sent as
follows:

```
0x52 => "R"
0x74657374696e67 => "testing"
9f5f6122d09c18bef1c9b96e773cf0b784198b70e4c3becbe4951d642ee4484c => parameters depend by each command
```


<a id="orgff8074f"></a>

## Process Listener message

It is defined at `peer/listener.go#157: process`. When receiving every
peer message, checks data validation. After decode message sent by
zeroMQ, an array of strings will be returned.

First string in array will be chain type.

```go
    theChain := string(data[0])
```

Second string in array will be operation action, third string in arary
will be parameters (if exists).

```go
    fn := string(data[1])
    parameters := data[2:]
```

<a id="orgb223f5c"></a>

### Server information ("I")

Returns server information with following format.

```go
    serverInfo{
        Version: lstn.version,
        Chain:   mode.ChainName(),
        Normal:  mode.Is(mode.Normal),
        Height:  block.GetHeight(),
    }
    result, err = json.Marshal(info)
```

`result` is converted into json format.


<a id="org668fca4"></a>

### Get block number ("N")

Returns block height.

```go
    blockNumber := block.GetHeight()
    result = make([]byte, 8)
    binary.BigEndian.PutUint64(result, blockNumber)
```

`result` is format into big-endian 64 bits.


<a id="org1c83679"></a>

### Get packed block ("B")

Returns block number specified by parameter. Return error if first parameter
length is not 8 bytes (64 bits).

```go
    if 1 != len(parameters) {
        err = fault.ErrMissingParameters
    } else if 8 == len(parameters[0]) {
        result = storage.Pool.Blocks.Get(parameters[0])
        if nil == result {
            err = fault.ErrBlockNotFound
        }
    } else {
        err = fault.ErrBlockNotFound
    }
```

<a id="org0cbef05"></a>

### Get block hash ("H")

Return block hash specified by parameters. Return error if first
parameter length is not 8 bytes (64 bits).

```go
    if 1 != len(parameters) {
        err = fault.ErrMissingParameters
    } else if 8 == len(parameters[0]) {
        number := binary.BigEndian.Uint64(parameters[0])
        d, e := block.DigestForBlock(number)
        if nil == e {
            result = d[:]
        } else {
            err = e
        }
    } else {
        err = fault.ErrBlockNotFound
    }
```

<a id="org6ac295e"></a>

### Register to another node ("R")

In order to register as new peer, some information are necessary
to provide including chain type, public key, listener ip/port,
timestamp. Before continuing the flow , all data will be checked
if it's valid or not.

```go
    var binTs [8]byte
    binary.BigEndian.PutUint64(binTs[:], uint64(ts.Unix()))

    _, err = socket.Send(fn, zmq.SNDMORE)
    logger.PanicIfError("Listener", err)
    _, err = socket.Send(chain, zmq.SNDMORE)
    logger.PanicIfError("Listener", err)
    _, err = socket.SendBytes(publicKey, zmq.SNDMORE)
    logger.PanicIfError("Listener", err)
    _, err = socket.SendBytes(listeners, zmq.SNDMORE)
    logger.PanicIfError("Listener", err)
    _, err = socket.SendBytes(binTs[:], 0)
    logger.PanicIfError("Listener", err)
```

<a id="orgc200c7d"></a>

### Default action

Default operation is to process received data. The function is
defined at `peer/process.go#23: processSubscription`.

Different data type will be passed, zero or more parameters may be
transfered.

```go
    func processSubscription(log *logger.L, command string, arguments [][]byte) {

        dataLength := len(arguments)
        switch string(command) { ... }
        ...
    }
```

Exact `command` listed as follows:

1.  Block ("block")

    Send block information in parameter.

```go
        if dataLength < 1 {
            log.Warnf("block with too few data: %d items", dataLength)
            return
        }
        log.Infof("received block: %x", arguments[0])
        if !mode.Is(mode.Normal) {
            err := fault.ErrNotAvailableDuringSynchronise
            log.Warnf("failed assets: error: %s", err)
        } else {
            messagebus.Bus.Blockstore.Send("remote", arguments[0])
        }
```

2.  Asset ("assets")

    Unpack message and cache it if message is an asset record. Detail
    asset information is unpacked by `peer/process.go#129:
         processAssets`.

```go
        transaction, n, err := transactionrecord.Packed(packed).Unpack(mode.IsTesting())
        ...
        switch tx := transaction.(type) {
        case *transactionrecord.AssetData:
            _, packedAsset, err := asset.Cache(tx)
            if nil != err {
                return err
            }
            if nil != packedAsset {
                ok = true
            }
```

    Incoming asset record is cached at `asset/asset.go#81: Cache`. The reason asset is
    cached because asset record should always comes with an issue, so asset
    cannot come alone.

    Asset record will be checked if already existed in cache pool.

```go
        switch tx := transaction.(type) {
        case *transactionrecord.AssetData:
            if tx.Name == asset.Name &&
                tx.Fingerprint == asset.Fingerprint &&
                tx.Metadata == asset.Metadata &&
                tx.Registrant.String() == asset.Registrant.String() {

                r.state = pendingState // extend timeout
                packedAsset = nil      // already seen
            } else {
                dataWouldChange = true
            }
        }
```

    After that, asset record will be put into a queue.

```go
        globalData.expiry.queue <- assetId
```

    `asset` object is initialized at `asset/asset.go#53: Initialise`,
    which will invoke a background job to process the queue just put:

```go
        globalData.background = background.Start(processes, globalData.log)
```

    The background job will call `Run` located at `asset/expiry.go#22:
         Run`. It setup default timeout to 72 hours at
    `constants/constants.go`. An asset record is cached for 72 hours
    for user to pay fee for miner.

3.  Issue ("issues")

    Incoming issue command is processed by `peer/process.go#169:
         processIssues`. After some data checkings, extract all issues
    inside parameter.

```go
        issues := make([]*transactionrecord.BitmarkIssue, 0, 100)
        for 0 != len(packedIssues) {
            transaction, n, err := packedIssues.Unpack(mode.IsTesting())
            if nil != err {
                return err
            }

            switch tx := transaction.(type) {
            case *transactionrecord.BitmarkIssue:
                issues = append(issues, tx)
                issueCount += 1
            default:
                return fault.ErrTransactionIsNotAnIssue
            }
            packedIssues = packedIssues[n:]
        }
```

    Actual issue processing logic is at `reservoir/issues.go#48: StoreIssues`.

    1.  Verify issue

        Make sure issues waiting for process has not exceed limits of
        `maximumIssues` (100).

        For all issues, check if asset record already exist because issue
        must go with asset record.

```go
            packedIssue, err := issue.Pack(issue.Owner)
            if nil != err {
                return nil, false, err
            }

            if !asset.Exists(issue.AssetId) {
                return nil, false, fault.ErrAssetNotFound
            }
```

        `asset.Exists` check asset info from both confirmed/cached list.

        After checking, create `txId`.

```go
            txId := packedIssue.MakeLink()
```

        From database table, issue record is placed inside `transaction`
        table, that's why naming of cache pool is `UnverifiedTxIndex`.

        If an issue is alread existed in the `UnverfiedTxIndex` pool, mark
        it as duplicate and do other checking.

```go
            if _, ok := cache.Pool.UnverifiedTxIndex.Get(txId.String()); ok {
                // if duplicate, activate pay id check
                duplicate = true
            }
```

        If an issue is already verifed, return error because it shouldn't happen.

```go
            if _, ok := cache.Pool.VerifiedTx.Get(txId.String()); ok {
                return nil, false, fault.ErrTransactionAlreadyExists
            }
```

        If an issue is already confirmed, return error because it
        shouldn't happen.

```go
            if storage.Pool.Transactions.Has(txId[:]) {
                return nil, false, fault.ErrTransactionAlreadyExists
            }
```

    2.  Compute payment info

        Generate payid, nonce, etc. Payment id is generated at
        `pay/payid.go#20: NewPayId`. Nonce is generated at
        `reservoir/paynonce.go#23: NewPayNonce`. Difficulty is generated
        at `reservoir/difficulty.go#35: ScaledDifficulty`.

```go
            payId := pay.NewPayId(separated)
            nonce := NewPayNonce()
            difficulty := ScaledDifficulty(count)

            result := &IssueInfo{
                Id:         payId,
                Nonce:      nonce,
                Difficulty: difficulty,
                TxIds:      txIds,
                Packed:     bytes.Join(separated, []byte{}),
                Payments:   nil,
            }
```

        If payment id is already generated, do nothing.

```go
            if _, ok := cache.Pool.UnverifiedTxEntries.Get(payId.String()); ok {
                globalData.log.Debugf("duplicate pay id: %s", payId)
                return result, true, nil
            }
```

    3.  Check for duplicated issue

        If a duplicate issue is detected but cannot found any duplicated
        payment info, return error.

```go
            if duplicate {
                globalData.log.Debugf("overlapping pay id: %s", payId)
                return nil, false, fault.ErrTransactionAlreadyExists
            }
```

    4.  Determine payment & block number for the issue

        If an issue record passed all previous checking, then it's time
        to find the block where asset record being placed. `GetNB` stands for
        get block number (NB).

```go
            assetBlockNumber := uint64(0)
            scan_for_one_asset:
            for _, issue := range issues {
                bn, t := storage.Pool.Assets.GetNB(issue.AssetId[:])
                if nil == t || 0 == bn {
                    assetBlockNumber = 0     // cannot determine a single payment block
                    break scan_for_one_asset // because of unconfirmed asset
                } else if 0 == assetBlockNumber {
                    assetBlockNumber = bn // block number of asset
                } else if assetBlockNumber != bn {
                    assetBlockNumber = 0     // cannot determin a single payment block
                    break scan_for_one_asset // because of multiple assets
                }
            }
```

            Get payment record from block.

```go
            if assetBlockNumber > genesis.BlockNumber { // avoid genesis block

                blockNumberKey := make([]byte, 8)
                binary.BigEndian.PutUint64(blockNumberKey, assetBlockNumber)

                p := getPayment(blockNumberKey)
                if nil == p { // would be an internal database error
                    globalData.log.Errorf("missing payment for asset id: %s", issues[0].AssetId)
                    return nil, false, fault.ErrAssetNotFound
                }

                result.Payments = make([]transactionrecord.PaymentAlternative, 0, len(p))
                // multiply fees for each currency
                for _, r := range p {
                    total := r.Amount * uint64(len(txIds))
                    pa := transactionrecord.PaymentAlternative{
                        &transactionrecord.Payment{
                            Currency: r.Currency,
                            Address:  r.Address,
                            Amount:   total,
                        },
                    }
                    result.Payments = append(result.Payments, pa)
                }
            }
```

    5.  Store issue into pool

        Generate issue data.

```go
            entry := &unverifiedItem{
                itemData: &itemData{
                    txIds:        txIds,
                    links:        nil,
                    assetIds:     uniqueAssetIds,
                    transactions: separated,
                    nonce:        nil,
                },
                //nonce:      nonce, // ***** FIX THIS: this value seems not used
                difficulty: difficulty,
                payments:   result.Payments,
            }
```

            If payment for an issue is already recieved, store payment and
        remove data from `OrphanPayment`.

```go
            if val, ok := cache.Pool.OrphanPayment.Get(payId.String()); ok {
                detail := val.(*PaymentDetail)

                if acceptablePayment(detail, result.Payments) {

                    for i, txId := range txIds {
                        cache.Pool.VerifiedTx.Put(
                            txId.String(),
                            &verifiedItem{
                                itemData:    entry.itemData,
                                transaction: separated[i],
                                index:       i,
                            },
                        )
                    }
                    cache.Pool.OrphanPayment.Delete(payId.String())
                    return result, false, nil
                }
            }
```

        If no payment found, put issues into cache pool.

```go
            for _, txId := range txIds {
                cache.Pool.UnverifiedTxIndex.Put(txId.String(), payId)
            }
            cache.Pool.UnverifiedTxEntries.Put(payId.String(), entry)
```

4.  Transfer ("transfer")

    Incoming transfer command is processed by `peer/process.go#215:
         processTransfer`. After some data checkings, store transfer record
    into pool by method `reservoir/transfer.go#27: StoreTransfer`.

    1.  Verify transfer record

        At `resovoir/transfer.go#127: verifyTransfer`.

        Check if any previous transaction exists.

```go
            __, previousPacked := storage.Pool.Transactions.GetNB(newTransfer.GetLink().Bytes())
            if nil == previousPacked {
                return nil, false, fault.ErrLinkToInvalidOrUnconfirmedTransaction
            }

            previousTransaction, _, err := transactionrecord.Packed(previousPacked).Unpack(mode.IsTesting())
            if nil != err {
                return nil, false, err
            }
```

        A new transfer record can happen on if previous transfer is in
        following types: issue, transfer, counter-signed transfer, old
        base data, block foundataion, block owner transfer

    2.  Generate transfer info

        A transfer info includes payment info, which is generated by
        previous transfer record.

```go
            packedTransfer := verifyResult.packedTransfer
            payId := pay.NewPayId([][]byte{packedTransfer})

            txId := verifyResult.txId
            link := transfer.GetLink()
            if txId == link {
                // reject any transaction that links to itself
                // this should never occur, but protect against this situation
                return nil, false, fault.ErrTransactionLinksToSelf
            }

            previousTransfer := verifyResult.previousTransfer
            ownerData := verifyResult.ownerData

            payments := getPayments(ownerData, previousTransfer)

            result := &TransferInfo{
                Id:       payId,
                TxId:     txId,
                Packed:   packedTransfer,
                Payments: payments,
            }
```

    3.  Already existed payment

        If a payment id is already existed in the cache pool, just
        return that payment info.

```go
            if val, ok := cache.Pool.UnverifiedTxEntries.Get(payId.String()); ok {
                entry := val.(*unverifiedItem)
                if nil != entry.payments {
                    result.Payments = entry.payments
                } else {
                    // this would mean that reservoir data is corrupt
                    logger.Panicf("StoreTransfer: failed to get current payment data for: %s  payid: %s", txId, payId)
                }
                return result, true, nil
            }
```

            If duplicated transfer already exist but not found unverified
        record, return error.

        Remove payment record from `OrphanPayment` is already recieved.

    4.  Wait for payment info

        For a payment that has not verifyed, put record into cache pool.

```go
            cache.Pool.PendingTransfer.Put(link.String(), txId)
            cache.Pool.UnverifiedTxIndex.Put(txId.String(), payId)
            cache.Pool.UnverifiedTxEntries.Put(
                payId.String(),
                &unverifiedItem{
                    itemData: transferredItem,
                    payments: payments,
                },
            )
```

5.  Proof ("proof")

    Incoming proof command is processed by `peer/process.go#247:
         processProof`. After some data checkings, if the record is a velid
    proof block, return nil, otherwise return error.

```go
        var payId pay.PayId
        nonceLength := len(packed) - len(payId) // could be negative
        if nonceLength < payment.MinimumNonceLength || nonceLength > payment.MaximumNonceLength {
            return fault.ErrInvalidNonce
        }
        copy(payId[:], packed[:len(payId)])
        nonce := packed[len(payId):]
        status := reservoir.TryProof(payId, nonce)
        if reservoir.TrackingAccepted != status {
            // pay id already processed or was invalid
            return fault.ErrPayIdAlreadyUsed
        }
        return nil
```

6.  RPC ("rpc")

    Incoming rpc command adds server to rpc list.

```go
        if announce.AddRPC(arguments[0], arguments[1], timestamp) {
            messagebus.Bus.Broadcast.Send("rpc", arguments[0:3]...)
        }
```

7.  Peer ("peer")

    Incoming peer command adds server to peer list.

```go
        if announce.AddPeer(arguments[0], arguments[1], timestamp) {
            messagebus.Bus.Broadcast.Send("peer", arguments[0:3]...)
        }
```

8.  Heart beat ("heart")

    Do nothing.


<a id="orgc437bbe"></a>

## Connector Peer Processing

Connector behavior can be viewed as a finite state machine, the
implementation is using for loop to execute.

```go
    for conn.runStateMachine() {
    }
```

Each state is integer value defined at `peer/connector.go#38`.

```go
    const (
        cStateConnecting   connectorState = iota // register to nodes and make outgoing connections
        cStateHighestBlock connectorState = iota // locate node(s) with highest block number
        cStateForkDetect   connectorState = iota // read block hashes to check for possible fork
        cStateFetchBlocks  connectorState = iota // fetch blocks from current or fork point
        cStateRebuild      connectorState = iota // rebuild database from fork point (config setting to force total rebuild)
        cStateSampling     connectorState = iota // signal resync complete and sample nodes to see if out of sync occurs
    )
```

`peer/connector.go#224: runStateMachine`. returns `true` or `false` used to
decide if state machine should stop or not.

Connector states are described as follows:


<a id="org8f4abb0"></a>

### Conneting

Check if every clint is connected and update client count. Stops
loop if minimum clients are connected (3).


<a id="org644afb4"></a>

### Hightest block

Get highest block from all clients. Go to next state
`cStateForkDetect` if there another node with larger block
height. Otherwise, stop loop.


<a id="orgc864745"></a>

### Fork detected

If other node has larger block height, go to rebuild state.

If current block height is larger than any other node's larget block
height, find the latest common block. If current block height gets
60 more blocks thatn latest common block, go to highest block
state, otherwise, remove old blocks.


<a id="org75dc618"></a>

### Fetch block

Fetch new blocks from other nodes.


<a id="org105a8e6"></a>

### Rebuild

Return to normal operation.


<a id="org86743a5"></a>

### Sampling

Check peers and block height to detect fork.
