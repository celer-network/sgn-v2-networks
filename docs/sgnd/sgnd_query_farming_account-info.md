## sgnd query farming account-info

query the info of a farming account

### Synopsis

Query the info of a farming account.

Example:
$ <appd> query farming account-info 0xab5801a7d398351b8be11c439e05c5b3259aec9b

```
sgnd query farming account-info [address] [flags]
```

### Options

```
      --height int      Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help            help for account-info
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
