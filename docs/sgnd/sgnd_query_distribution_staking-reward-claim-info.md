## sgnd query distribution staking-reward-claim-info

query the staking reward claim info of an account

### Synopsis

Query the staking reward claim info of an account.

Example:
$ <appd> query staking-reward-claim-info 0xd0f2596d700c9bd4d605c938e586ec67b01c7364

```
sgnd query distribution staking-reward-claim-info [delegator-address] [flags]
```

### Options

```
      --height int      Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help            help for staking-reward-claim-info
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
