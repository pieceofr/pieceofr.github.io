Table of Contents

1.  [RPC flow](#orgcc3b947)
    1.  [Initialization](#org9e7029b)
    2.  [RPC protocol](#orgca4c6bd)
    3.  [Environment (optional)](#org456c295)
    4.  [Available methods](#orga8399f5)
        1.  [Assets.Get](#orgb94d729)
        2.  [Bitmark.Provenance](#orgf9d332b)
        3.  [Bitmark.Transfer](#orgc7f2626)
        4.  [Bitmarks.Create](#org7eb52a7)
        5.  [Bitmarks.Proof](#org7db3cd5)
        6.  [BlockOwner.Transfer](#org7b76ce0)
        7.  [Node.Info](#org54faab5)
        8.  [Node.List](#org54faab6)
        9.  [Transaction.Status](#orgefb9638)


<a id="orgcc3b947"></a>

# RPC flow

All following paragraphs are describing `bitmarkd` version `8.2`
commit `b013d5e`


<a id="org9e7029b"></a>

## Initialization

When `bitmarkd` starts, it does some initialization processes including rpc.

rpc is setup in the `command/bitmarkd/main.go#294:
   rpc.Initialise`. In this method, both rpc and https are
initialized. This package has global data structure as following:

```go
    type rpcData struct {
        sync.RWMutex // to allow locking

        log *logger.L // logger

        listener *listener.MultiListener

        httpServer *httpHandler

        // set once during initialise
        initialised bool
    }
```

The `RWMutex` states for read/write data locking.

The rpc listeners are type of `MultiListener` include `channels`:

```go
    type MultiListener struct {
        state            int32
        log              *logger.L
        tlsConfiguration *tls.Config
        callback         Callback
        limiter          *Limiter
        listeners        []*channel
    }
```

From `rpc/setup.go#113: initialiseRPC` it specifies the limit of rpc
connection count by `maximum_connections` in configuration file.

`rpc/setup.go#138: NewMultiListener` specifies that each rpc
connected by tcp protocol, and it also specifies the callback
function in its last parameters. The `Callback` is a function at
`rpc/server.go#28: Callback`.

```go
    codec := jsonrpc.NewServerCodec(conn)
    defer codec.Close()
    server.ServeCodec(codec)
```

When running `server.ServeCodec`, it will use go native package
`rpc/server.go#461: ServeCodec`, it parses request by
`rpc/server.go#587: readRequestHeader`, service name and method name
is determiined by last dot.

```go
    dot := strings.LastIndex(req.ServiceMethod, ".")
    serviceName := req.ServiceMethod[:dot]
    methodName := req.ServiceMethod[dot+1:]
```

`rpc/setup.go#165: createRPCServer` registers all available rpc
methods at `rpc/setup.go#293`

```go
    server.Register(assets)
    server.Register(bitmark)
    server.Register(bitmarks)
    server.Register(owner)
    server.Register(node)
    server.Register(transaction)
    server.Register(blockOwner)
```

rpc server starts to listen by code `rpc/setup.go#172:
   globalData.listener.Start`, the actual start of rpc is at
`listener/multiListener.go#118: StartListening`.

```go
    address, err := net.ResolveTCPAddr(tcpVersion, listenAddress)
    // ...
    listeningSocket, err := net.ListenTCP(tcpVersion, address)
    // ...
    go server(&listener)
```

Last line starts a goroutine to run `server`,the source code at
`listener/listener.go#157: server`.


<a id="orgca4c6bd"></a>

## RPC protocol

go natively support `json-rpc` version 1.0, the protocol can be
referenced from [here](https://www.jsonrpc.org/specification_v1)

Simple `json-rpc` includes `id`, `method`, and `params`. `method` is the
function wants to be executed, `params` is the arguments to that function.


<a id="org456c295"></a>

## Environment (optional)

If anyone wants to try for the rpc connection, `bitmarkd` needs to be
started in advance. Following command uses simple shell script
using `openssl` to communicate.

Also, it assumes `bitmarkd` is listening local port `2130`.

```shell
    (echo '{"id":"1","method":"Bitmark.Provenance","params":[{"count":20,"txId":"2dc8770718b01f0205ad991bfb4c052f02677cff60e65d596e890cb6ed82c861"}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="orga8399f5"></a>

## Available methods

All methods are named as `Pakcage-name.method-name`, for example,
`Assets.Get` stands for method `Get` in file `assets.go`.


<a id="orgb94d729"></a>

### Assets.Get

This method is in file `rpc/assets.go#81: Get`. It is to get asset information.

Input format `AssetGetArguments` has an array of strings to
represent asset fingerprints (`fingerprints`).

```go
    type AssetGetArguments struct {
        Fingerprints []string `json:"fingerprints"`
    }
```

Example rpc shell comand as follows:

```shell
          (echo '{"id"
    :"1","method":"Assets.Get","params":[{"fingerprints": ["01c7499e5d27880da75a3747340aa1cb2f11fa023f7aa1eb10acbf28e447aefd366759092de7a31fdfe89fcb88ecc3c90c0e067484184f41a8e3043e8aa4732f00", "01f9c2eed799128d331c0e60b07d99bffa299b100eea7bb4a410d8f2ee2a04218623f2dbd1a53f0fe08f9cda131ecff213889cbd2cf5c8e53100581ff00f6270c1"]}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="orgf9d332b"></a>

### Bitmark.Provenance

This method is in file `rpc/bitmark.go#120: Provenance`. It is to
get provenance of a bitmark.

Input format `ProvenanceArguments` includes a string to
represent transaction id (`txId`), and the number of maximum records
to return (`count`)

```go
    type ProvenanceArguments struct {
        TxId  merkle.Digest `json:"txId"`
        Count int           `json:"count"`
    }
```

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Bitmark.Provenance","params":[{"count":20,"txId":"2dc8770718b01f0205ad991bfb4c052f02677cff60e65d596e890cb6ed82c861"}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="orgc7f2626"></a>

### Bitmark.Transfer

This method is in file `rpc/bitmark.go#36: Transfer`. It is used to
transfer bitmark.

Input format `transactionrecord.BitmarkTransferCountersigned`
includes previous record (`link`), options escrow payment address
(`escrow`), accoutn owner (`owner`), hex string of account signature (`signature`),
and hex string of counter-signature (`countersignature`).

```go
    type BitmarkTransferCountersigned struct {
        Link             merkle.Digest     `json:"link"`             // previous record
        Escrow           *Payment          `json:"escrow"`           // optional escrow payment address
        Owner            *account.Account  `json:"owner"`            // base58: the "destination" owner
        Signature        account.Signature `json:"signature"`        // hex: corresponds to owner in linked record
        Countersignature account.Signature `json:"countersignature"` // hex: corresponds to owner in this record
    }
```

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Bitmark.Transfer","params":[{"link":"1bebd06c8ecb8b11ea93e93c9d38b7f6d7dfdf015530819015172cf51c7f33f7", "owner": "eZpG6Wi9SQvpDatEP7QGrx6nvzwd6s6R8DgMKgDbDY1R5bjzb9", "signature": "a3e456a31a4a64962a32bcbf6549d14134deeb5d87285a04c648355eb9e59d938f8ab440d2b50c781baf2c1a5a2112c2167301bb128c8f850a9d54f3b27c5a08"}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="org7eb52a7"></a>

### Bitmarks.Create

This method is in file `rpc/bitmarks.go#57: Create`. It is used to
create bitmark.

Input format `CreateArguments` includes array of asset data (`assets`)
and array of issues (`issues`)

```go
    type CreateArguments struct {
        Assets []*transactionrecord.AssetData    `json:"assets"`
        Issues []*transactionrecord.BitmarkIssue `json:"issues"`
    }

    type AssetData struct {
        Name        string            `json:"name"`        // utf-8
        Fingerprint string            `json:"fingerprint"` // utf-8
        Metadata    string            `json:"metadata"`    // utf-8
        Registrant  *account.Account  `json:"registrant"`  // base58
        Signature   account.Signature `json:"signature"`   // hex
    }

    type BitmarkIssue struct {
        AssetId   AssetIdentifier   `json:"assetId"`   // link to asset record
        Owner     *account.Account  `json:"owner"`     // base58: the "destination" owner
        Nonce     uint64            `json:"nonce"`     // to allow for multiple issues at the same time
        Signature account.Signature `json:"signature"` // hex: corresponds to owner in linked record
    }
```

The `Metadata` data has two fields for each record, `key` and `value`,
separated by `\u0000`, e.g. `k1\u0000v1\u0000k2\u0000v2` means `key` k1 has
`value` v1.

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Bitmarks.Create","params":[{"assets": [{"name": "asset", "fingerprint": "01840006653e9ac9e95117a15c915caab81662918e925de9e004f774ff82d7079a40d4d27b1b372657c61d46d470304c88c788b3a4527ad074d1dccbee5dbaa99a", "metadata": "k1\u0000v1\u0000k2\u0000v2", "registrant": "e1pFRPqPhY2gpgJTpCiwXDnVeouY9EjHY6STtKwdN6Z4bp4sog", "signature": "dc9ad2f4948d5f5defaf9043098cd2f3c245b092f0d0c2fc9744fab1835cfb1ad533ee0ff2a72d1cdd7a69f8ba6e95013fc517d5d4a16ca1b0036b1f3055270c"}], "issues": [{"assetId": "3c50d70e0fe78819e7755687003483523852ee6ecc59fe40a4e70e89496c4d45313c6d76141bc322ba56ad3f7cd9c906b951791208281ddba3ebb5e7ad83436c", "owner": "e1pFRPqPhY2gpgJTpCiwXDnVeouY9EjHY6STtKwdN6Z4bp4sog", "nonce": 4, "signature": "6ecf1e6d965e4364321596b4675950554b3b8f1b40be3deb64306ddf72fef09f3c6bcebd6375925a51b984f56ec751a54c88f0dab56b3f69708a7b634c428a0a"}]}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="org7db3cd5"></a>

### Bitmarks.Proof

This method is in the file `rpc/bitmarks.go#147: Proof`. It is used
to validate payment.

Input format `ProofArguments` includes payment id (`payId`) and
nonce (`nonce`).

```go
    type ProofArguments struct {
        PayId pay.PayId `json:"payId"`
        Nonce string    `json:"nonce"`
    }
```

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Bitmarks.Proof","params":[{"payId":"2ad3ba0b28fe98716bb8d87169a952eebfc4aff96b4f9eb7de7d4c71c7acee79", "nonce": "c114fa516a98c3de"}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="org7b76ce0"></a>

### BlockOwner.Transfer

This method is in the file `rpc/blockowner.go#72: Transfer`. It is
used to transfer blocks.

Input format `transactionrecord.BlockOwnerTransfer` includes some
block owner information as follows:

```go
    type BlockOwnerTransfer struct {
        Link             merkle.Digest     `json:"link"`             // previous record
        Escrow           *Payment          `json:"escrow"`           // optional escrow payment address
        Version          uint64            `json:"version"`          // reflects combination of supported currencies
        Payments         currency.Map      `json:"payments"`         // require length and contents depend on version
        Owner            *account.Account  `json:"owner"`            // base58
        Signature        account.Signature `json:"signature,"`       // hex
        Countersignature account.Signature `json:"countersignature"` // hex: corresponds to owner in this record
    }
```

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"BlockOwner.Transfer","params":[{"link":"1bebd06c8ecb8b11ea93e93c9d38b7f6d7dfdf015530819015172cf51c7f33f7", "version": 5, "payments": ["1": "BTC"], "owner": "eZpG6Wi9SQvpDatEP7QGrx6nvzwd6s6R8DgMKgDbDY1R5bjzb9", "signature": "a3e456a31a4a64962a32bcbf6549d14134deeb5d87285a04c648355eb9e59d938f8ab440d2b50c781baf2c1a5a2112c2167301bb128c8f850a9d54f3b27c5a08"}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="org54faab5"></a>

### Node.Info

This method is in file `rpc/node.go#77: Info`. It is used to get
node info.

There no need to have any parameter for this method.

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Node.Info","params":[{}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="org54faab6"></a>

### Node.List

Lists the RPC services available in the network.

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Node.List","params":[{}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```

<a id="orgefb9638"></a>

### Transaction.Status

This method is in file `rpc/transaction.go#32: Status`. It is used
to get transaction status.

Input format `TransactionArguments` includes string (`txId`)

```go
    type TransactionArguments struct {
        TxId merkle.Digest `json:"txId"`
    }
```

Example rpc shell command as follows:

```shell
    (echo '{"id":"1","method":"Transaction.Status","params":[{"txId":"2dc8770718b01f0205ad991bfb4c052f02677cff60e65d596e890cb6ed82c861"}]}'; sleep 1; echo Q) | openssl s_client -connect 127.0.0.1:2130 -quiet
```