## sgnd query staking syncer-instructions

Query syncer instructions of all syncer candidates

```
sgnd query staking syncer-instructions [flags]
```

### Options

```
      --height int      Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help            help for syncer-instructions
      --node string     <host>:<port> to Tendermint RPC interface for this chain (default "tcp://localhost:26657")
  -o, --output string   Output format (text|json) (default "text")
```

### Options inherited from parent commands

```
      --chain-id string   The network chain ID
```

### SEE ALSO

* [sgnd query staking](sgnd_query_staking.md)	 - Querying commands for the staking module

###### Auto generated by spf13/cobra
