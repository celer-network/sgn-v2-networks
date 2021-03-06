## sgnd query farming accounts-staked-in

query the addresses of accounts staked in a pool

### Synopsis

Query all the addresses of accounts that have staked tokens in a specific pool.

Example:
$ <appd> query farming accounts-staked-in cbridge-DAI/1

```
sgnd query farming accounts-staked-in [pool-name] [flags]
```

### Options

```
      --height int      Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help            help for accounts-staked-in
      --node string     <host>:<port> to Tendermint RPC interface for this chain (default "tcp://localhost:26657")
  -o, --output string   Output format (text|json) (default "text")
```

### Options inherited from parent commands

```
      --chain-id string   The network chain ID
```

### SEE ALSO

* [sgnd query farming](sgnd_query_farming.md)	 - Querying commands for the farming module

###### Auto generated by spf13/cobra
