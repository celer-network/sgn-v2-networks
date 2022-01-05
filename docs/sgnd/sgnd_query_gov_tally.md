## sgnd query gov tally

Get the tally of a proposal vote

### Synopsis

Query tally of votes on a proposal. You can find
the proposal-id by running "<appd> query gov proposals".

Example:
$ <appd> query gov tally 1

```
sgnd query gov tally [proposal-id] [flags]
```

### Options

```
      --height int   Use a specific height to query state at (this can error if the node is pruning state)
  -h, --help         help for tally
```

### Options inherited from parent commands

```
      --chain-id string   The network chain ID
```

### SEE ALSO

* [sgnd query gov](sgnd_query_gov.md)	 - Querying commands for the governance module

###### Auto generated by spf13/cobra