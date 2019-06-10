### account
+ Account is to do account decoding from base58

    + Account decoding functions

    ```
    AccountFromBase58(accountBase58Encoded string) (*Account, error)
    AccountFromBytes(accountBytes []byte) (*Account, error)
    ```
    
+ It also has a section to handle private key to decode variant from configuration file.

    + Private key decoding fuctions
    
    ```
    func PrivateKeyFromBase58(privateKeyBase58Encoded string) (*PrivateKey, error)
    func PrivateKeyFromBytes(privateKeyBytes []byte) (*PrivateKey, error)
    ```
        
    + Private keys setting in bitmarkd.conf 

    ```
    proofing {
      public_key = proof.public
      private_key = proof.private
      signing_key = proof.sign
    }
    ```

### announce
+ Read initial DNS records to find out starting place for ips and ports in the network
    + Lookup TXT Record of DNS records
      ```net.LookupTXT(nodesDomain)```  
+ Background process to announce ip and ports to the network

### asset
  + memory buffer to store pending assets
    ```
      type cacheData struct {
        packed transactionrecord.Packed // data
        state  assetState // used to detect expired/verified items
    }
    ```
  + background process to expire pending asset which does not verify within 3 days
  ```expiry.go```
  
### avl
 + A balanced binary tree to store nodes in the network

### background
 + Factory method to create background processes
 + background process can be Start() and Stop()

 ```
     func Example() {

        proc := &theState{
            count: 10,
        }

        // list of background processes to start
        processes := background.Processes{
            proc,
        }

        p := background.Start(processes, nil)
        time.Sleep(time.Second)
        p.Stop()
    }
 ```
 
### block
+ subroutine to recieve blocks from broadcasting queue
+ broadcasting to other peers if block is valid
+ Decoding Header and check difficulty etc. to make sure block is valid
+ Handle all transactions of a block and save block to database

### blockdigest
+ High level interface to the argon2 for hashing block
+ Utilities to handle digests ie. comparing difficulty or  marshal text

### blockrecord

+ header.go : actual struct of a block
```
    // the types here must match Bitcoin header types
    type Header struct {
        Version          uint16                 `json:"version"`
        TransactionCount uint16                 `json:"transactionCount"`
        Number           uint64                 `json:"number,string"`
        PreviousBlock    blockdigest.Digest     `json:"previousBlock"`
        MerkleRoot       merkle.Digest          `json:"merkleRoot"`
        Timestamp        uint64                 `json:"timestamp,string"`
        Difficulty       *difficulty.Difficulty `json:"difficulty"`
        Nonce            NonceType              `json:"nonce"`
    }
```
+ Pack and Unpack header
+ Convert nonce 

### blockring 
+ the method is used to proof an issues
+ give a nonce for the free payment issue. 
+ How do we preventing someone create thounds of issues and flooding it into the block.
+ hash cash
    + give someone a random hash number, hash that with the issue data
    + give an issue difficulty.

### c-library

hold libucl /libagon2 in c code for building

### cache
store pending transcation and verify issue in two memory buffers

### chain
chain named possible network and validation function
 ```
    // names of all chains
    const (
	    Bitmark = "bitmark" // the main network
	    Testing = "testing" // the global testing network which mostly mimics the main network
	    Local   = "local"   // a local testing network
    )
 ```

### command
All main programs are in command folder
+ bitmarkd : main blockchain program
        main.go/configuration.go
+ bitmark-cli : command line system sending rpc to bitmarkd and printout json response
+ recorderd : monitor pushish port in bitmarkd for a new block and hash it. It submit the block if it find a suitable nounce. It does proof of work.
    
### configuration

Each main programs in command modules will call the configuration file and send the root configuration file to the module. The module will do all the configurations.

### constants

Some constants related to peer networking.
```
ReservoirTimeout = 72 * time.Hour
AssetTimeout = ReservoirTimeout + time.Hour
RebroadcastInterval = 15 * time.Minute
```


### counter

go atomic functions to increament and decreament of global counter.

### currency

+ Verify the bitcoin and litecoin address. 
+ Do Satoshi conversion. 
+ Our testing network accpets  bitcoin/litecoin testnet address and bitmark network accepts bitcoin/litecoin mainnet address.

### debian

debian is for building bitmarkd and recorderd in debain/ubuntu. Debian has decided to go to build go dependencies in seperated packages. FreeBSD has decided not to. The purpose of this foldr is to build bitmarkd in specified packages. In this package, there are install scripts but build script is missing.  

### difficulty

Difficulty is the number which you compare against the block hash. It treats block hash as a 256 bit number as a kind of floating value. Difficulty module provide convertion from floating point (i.e value 1.0)  into the equivalent 256 bits number and then compare functions so you can compare blockhash againt current difficulty.

#### filter submodule 

Difficulty has a submodule called filters which try to do continously modification with difficult value base on how fast a block could be generated. The filter also can back thing up very quickly if somebody disconnected a bounch of proof of work in the network. Bitcoin takes two weeks to recover;however the two weeks is in terms of number of blocks. If somebody in bitcoin disconnected half of proof of work, it will take 4 weeks for Bitcoin to recover to 10 minutes per block. 

Filter submodule improve this result by continously monitoring block time for past few hours and adjust difficulty quickly. It improve time from 4 weeks to 8 hours compared with Bitcoin. 

### doc

Go standard directory to do documentation.

### fault

List of standard error messages.

### genesis

The place to generate genesis block/ first block for livenet and testnet. 

### keypair

This is handling the seed, decoding the account private key from the configure file so that bitmark can sign the foundation record. 

### merkle

A merkle tree can reduce a set of transcations to a single hash and be able to eliminate individual transcation from the verification process. We used SHA3 algorithem for hashing.

### messagebus
Messagebus is a set of queues inside bitmarkd processes. The purpose is to communicate different parts in the program.

#### List of queues
 
```
type busses struct {
    Broadcast  *BroadcastQueue `size:"1000"` // to broadcast to other nodes
    Connector  *Queue          `size:"50"`   // to control connector
    Announce   *Queue          `size:"50"`   // to control the announcer
    Blockstore *Queue          `size:"50"`   // to sequentially store blocks
    TestQueue  *Queue          `size:"50"`   // for testing use
}

``` 

+ broadcasting queue : creates multiple go channels so that all listeners in program can recieve messages
+ test queue : used for unit test for the module

### mode

+ Constants of mode setting
```
const (
	Stopped Mode = iota // the node is stopped
	Resynchronise // (remote_block_height - local_block_height) >= 2
	Normal        // (remote_block_height - local_block_height) < 2
	maximum
)
```
+ Mode set/change

### ownership

+ List of ownership of bitmarks
+ when bitmark transfer from one owner to another, this module can update the ownership of a bitmark
+ It has a routine of handling state change of ownership.

### pay

Hold payment id paid to writer. Something to write into currency blockchain so we can identify what is paying for.

### payment

Payment is actual payment processing. It recieve update from blockchain and find out what is pay for examing payid in the op_return of the currency. If it was a pending transcation without payid, mark it verify. If it was not, it saves its payid, just in case transaction occures later. It also has a background process monitor bitcoin and litecoin for incomming payments.

### peer

+ peer module handle all node to node communication. It uses zeromq REQ/RES socket.
![](https://i.imgur.com/yJI6WLh.png)

+ Connection limit currently is set to 10
+ Upstream submodule is the actual connection client
    + background process for each upstream
    ```func upstreamRunner(u *Upstream, shutdown <-chan struct{})```
+ rpc interface to fetch information


### proof

It is the interface to the recorderd. It has two pars
+ publishing part is where publish blocks availble to be mined. It does that every 30 secs. 
+ submission parts is where to submit block with nonce which may be work. 

### publish
publish is the interface to updaterd. **Updaterd** takes transcations and blocks and store them into the database. The transcation is pending transcation and the block is the confirm blocks. The purpose of publish with updaterd is to maintain externerl database of blockchain.

### reservoir

Reservoir is the memory storage to hold and verify the pending transcations. It moves from reservoir after transcation is verified. 

### rpc
The rpc moudle is a JSON-RPC which is a remote procedure call protocol encoded in JSON

+ The format rpc is **type.method** 
    ```
    Assets.Get
    Bitmarks.Create
    ```
    + example code in bitmark-cli
    ```
    err := client.client.Call("Bitmarks.Create", &args, &reply)
    ```
+ It also has a http rpc that can be query by curl

### storage
leveldb is a single key and value database. storage module use prefix to create leveldb tables and make it easier to access the tables.
 ```
 type pools struct {
	Blocks            *PoolHandle `prefix:"B" database:"blocks"`
	BlockOwnerPayment *PoolHandle `prefix:"H" database:"index"`
	BlockOwnerTxIndex *PoolHandle `prefix:"I" database:"index"`
	Assets            *PoolNB     `prefix:"A" database:"index"`
	Transactions      *PoolNB     `prefix:"T" database:"index"`
	OwnerCount        *PoolHandle `prefix:"N" database:"index"`
	Ownership         *PoolHandle `prefix:"K" database:"index"`
	OwnerDigest       *PoolHandle `prefix:"D" database:"index"`
	TestData          *PoolHandle `prefix:"Z" database:"index"`
}

 ```
 
 + It has several functions to operate database using prefix
 ```
 func (p *PoolHandle) Put(key []byte, value []byte)
 func (p *PoolHandle) Delete(key []byte)
 func (p *PoolHandle) Get(key []byte) []byte 
 ```
 + doc.go contain information the structure of table


### transactionrecord
+ All the structures for every records we can have in blockchain in transcation.go.
+  Pack and unpack routines to transfer them from binary data to go structures.
```
// the unpacked Proofer Data structure (OBSOLETE)
// this is first tx in every block and can only be used there
type OldBaseData struct {
	Currency       currency.Currency `json:"currency"`       // utf-8 → Enum
	PaymentAddress string            `json:"paymentAddress"` // utf-8
	Owner          *account.Account  `json:"owner"`          // base58
	Nonce          uint64            `json:"nonce,string"`   // unsigned 0..N
	Signature      account.Signature `json:"signature,"`     // hex
}

// the unpacked Asset Data structure
type AssetData struct {
	Name        string            `json:"name"`        // utf-8
	Fingerprint string            `json:"fingerprint"` // utf-8
	Metadata    string            `json:"metadata"`    // utf-8
	Registrant  *account.Account  `json:"registrant"`  // base58
	Signature   account.Signature `json:"signature"`   // hex
}

// the unpacked BitmarkIssue structure
type BitmarkIssue struct {
	AssetId   AssetIdentifier   `json:"assetId"`   // link to asset record
	Owner     *account.Account  `json:"owner"`     // base58: the "destination" owner
	Nonce     uint64            `json:"nonce"`     // to allow for multiple issues at the same time
	Signature account.Signature `json:"signature"` // hex: corresponds to owner in linked record
}

// optional payment record
type Payment struct {
	Currency currency.Currency `json:"currency"`      // utf-8 → Enum
	Address  string            `json:"address"`       // utf-8
	Amount   uint64            `json:"amount,string"` // number as string, in terms of smallest currency unit
}

// a single payment possibility - for use in RPC layers
// up to entries:
//   1. issue block owner payment
//   2. last transfer block owner payment (can merge with 1 if same address)
//   3. optional transfer payment
type PaymentAlternative []*Payment

// to access field of various transfer types
type BitmarkTransfer interface {
	Transaction
	GetLink() merkle.Digest
	GetPayment() *Payment
	GetOwner() *account.Account
	GetCurrencies() currency.Map
	GetSignature() account.Signature
	GetCountersignature() account.Signature
}

// the unpacked BitmarkTransfer structure
type BitmarkTransferUnratified struct {
	Link      merkle.Digest     `json:"link"`      // previous record
	Escrow    *Payment          `json:"escrow"`    // optional escrow payment address
	Owner     *account.Account  `json:"owner"`     // base58: the "destination" owner
	Signature account.Signature `json:"signature"` // hex: corresponds to owner in linked record
}

// the unpacked Countersigned BitmarkTransfer structure
type BitmarkTransferCountersigned struct {
	Link             merkle.Digest     `json:"link"`             // previous record
	Escrow           *Payment          `json:"escrow"`           // optional escrow payment address
	Owner            *account.Account  `json:"owner"`            // base58: the "destination" owner
	Signature        account.Signature `json:"signature"`        // hex: corresponds to owner in linked record
	Countersignature account.Signature `json:"countersignature"` // hex: corresponds to owner in this record
}

// the unpacked Proofer Data structure
// this is first tx in every block and can only be used there
type BlockFoundation struct {
	Version   uint64            `json:"version"`      // reflects combination of supported currencies
	Payments  currency.Map      `json:"payments"`     // contents depend on version
	Owner     *account.Account  `json:"owner"`        // base58
	Nonce     uint64            `json:"nonce,string"` // unsigned 0..N
	Signature account.Signature `json:"signature,"`   // hex
}

// the unpacked Block Owner Transfer Data structure
// forms a chain that links back to a foundation record which has a TxId of:
// SHA3-256 . concat blockDigest leBlockNumberUint64
type BlockOwnerTransfer struct {
	Link             merkle.Digest     `json:"link"`             // previous record
	Escrow           *Payment          `json:"escrow"`           // optional escrow payment address
	Version          uint64            `json:"version"`          // reflects combination of supported currencies
	Payments         currency.Map      `json:"payments"`         // require length and contents depend on version
	Owner            *account.Account  `json:"owner"`            // base58
	Signature        account.Signature `json:"signature,"`       // hex
	Countersignature account.Signature `json:"countersignature"` // hex: corresponds to owner in this record
}
````



### util
It gets a bounch of misc utilities. 
+ base58.go
+ canoical.go : convert ip and port
+ fetcher.go : fetch json
+ formatbytes.go
+ path.go
+ variant.go : convert variant and several operations

### zmqutil

Interface of on top zeromq. It is for easier to find and monitor sockets.


###### tags: `bitmarkd` `documentation` `modules` `overview`