## sgnd query staking delegation

Query a delegation based on delegator and validator's Ethereum addresses

### Synopsis

Query delegations for an individual delegator on an individual validator.

Example:
$ <appd> query staking delegation 0x00078b31fa8b29a76bce074b5ea0d515a6aeaee7 0xd0f2596d700c9bd4d605c938e586ec67b01c7364

```
sgnd query staking delegation [delegator-eth-addr] [validator-eth-addr] [flags]
```

### Options

```
      --height int      Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help            help for delegation
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