## sgnd query distribution commission

Query distribution validator commission

### Synopsis

Query validator commission rewards from delegators to that validator.

Example:
$ <appd> query distribution commission 0x00078b31fa8b29a76bce074b5ea0d515a6aeaee7

```
sgnd query distribution commission [validator] [flags]
```

### Options

```
      --height int      Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help            help for commission
      --node string     <host>:<port> to Tendermint RPC interface for this chain (default "tcp://localhost:26657")
  -o, --output string   Output format (text|json) (default "text")
```

### Options inherited from parent commands

```
      --chain-id string   The network chain ID
```

### SEE ALSO

* [sgnd query distribution](sgnd_query_distribution.md)	 - Querying commands for the distribution module

###### Auto generated by spf13/cobra
