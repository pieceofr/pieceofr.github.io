## Config file format

The configuration language is Lua. [See more details](https://github.com/bitmark-inc/bitmarkd/wiki/Configuring-bitmarkd#config-file-format).

### Sample

See the [config example](https://github.com/bitmark-inc/bitmarkd/blob/master/command/recorderd/recorderd.conf.sample).

## Base Section

`M.data_directory`, `M.pidfile` and `M.chain` are similar settings to [bitmarkd](https://github.com/bitmark-inc/bitmarkd/wiki/Configuring-bitmarkd#base-section).

`M.threads` (optional, default = number of CPUs): Number of background hashing threads.

## `M.peering` Section

`public_key` and `private_key`: Required keys to construct secure connections with *bitmarkd*.

`connect`: The *bitmarkd* nodes to connect to. The settings should correspond to the [proof section](#Mproofing-Section-optional) of *bitmarkd*.

## `M.logging` Section

The configuration is similar to [bitmarkd logging](https://github.com/bitmark-inc/bitmarkd/wiki/Configuring-bitmarkd#mlogging-section).

The configurable modules are `main`, `proofer-N`, `submitter-N`, `subscriber-N` (N is in the range [1..m] if there are m hashing threads). 