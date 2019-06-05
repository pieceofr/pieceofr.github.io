

# Quick setup (for testnet)

The assumption here is that the system is running in a terminal under a window manager that sets up
the XDG_* environment.  Some system running under FreeBSD or Linux running any desktop should work.
How ever if you are just run on the console with nothing much installed (possible via ssh) then if you may have to `export XDG_CONFIG_HOME="${HOME}/.config` at the start of the session (check the output of `printenv` to see if XDG_* are already setup)

~~~
% bitmark-cli --network=testing --identity='just-me' setup --testnet --description='my testnet account' --connect='node-d3.test.bitmark.com:2130'
Set identity password(length >= 8): 
Verify password: 
~~~

This will create the initial testnet configuration and will contain JSON with encrypted hex data fields:
~~~
% ls .config/bitmark-cli 
testing-bitmark-cli.json

% cat .config/bitmark-cli/testing-bitmark-cli.json 
{
  "default_identity": "just-me",
  "testnet": true,
  "connect": "node-d3.test.bitmark.com",
  "identities": [
    {
      "name": "just-me",
      "description": "my testnet account",
      "public_key": "XX..XX",
      "private_key": "XX..XX",
      "seed": "XX..XX",
      "private_key_config": {
        "salt": "XX..XX"
      }
    }
  ]
}
~~~

# Check network status

The command below will display a JSON block showing the state of the network

~~~
% bitmark-cli --network=testing --identity='just-me' bitmarkInfo
~~~

# Help for commands

List all commands:
~~~
% bitmark-cli --network=testing --identity='just-me' help
~~~

List specific command
~~~
% bitmark-cli --network=testing --identity='just-me' help create
~~~

# Creating an initial free bitmark
~~~
% bitmark-cli --network=testing --identity='just-me' fingerprint --file=my-asset.pdf
{
  "file_name": "my-asses.pdf",
  "fingerprint": "XX..FP..XX"
}
% bitmark-cli --network=testing --identity='just-me'  --asset="my PDF file" --metadata='desc\u0000some discription\u0000version\u00001.7.9 --fingerprint="XX..FP..XX" --zero
~~~

# Other networks

Notes
1. that all networks can coexist since they have different config files
2. when configuring for bitmark network be sure to add --livenet to setup command.  (This will corrected in future update)


## The main bitmark network

Note: use a strong password here!

~~~
% bitmark-cli --network=bitmark --identity='just-me' setup --livenet --description='my bitmark account' --connect='node-d4.live.bitmark.com'
Set identity password(length >= 8): 
Verify password: 
~~~

# For local testing

This is for a group of bitmarkd programmd runn over localhost
~~~
% bitmark-cli --network=local --identity='just-me' setup --testnet --description='my local account' --connect='127.0.0.1:2130'
Set identity password(length >= 8): 
Verify password: 
~~~
