## sgnd tx gov submit-proposal farming-adjust-reward

Submit an AdjustRewardProposal

### Synopsis

Submit an AdjustRewardProposal along with an initial deposit.
The proposal details must be supplied via a JSON file.

Example:
$ <appd> gov submit-proposal farming-adjust-reward <path/to/proposal.json> --from=<key_or_address>

Where proposal.json contains:

{
  "title": "cbridge-DAI/1 reward adjustment",
  "description": "Add DAI reward for cbridge-DAI/1 and adjust CELR reward",
  "pool_name": "cbridge-DAI/1",
  "reward_adjustment_inputs": [
    {
      "add_amount": {
        "denom": "CELR/1",
        "amount": "100000000000000000000000"
      },
      "reward_start_block_delay": 0,
      "new_reward_amount_per_block": "500000000000000000"
    },
    {
      "add_amount": {
        "denom": "USDT/1",
        "amount": "100000000000"
      },
      "reward_start_block_delay": 3,
      "new_reward_amount_per_block": "1000000"
    }
  ],
  "deposit": "10000000CELR/stake"
}

```
sgnd tx gov submit-proposal farming-adjust-reward [proposal-file] [flags]
```

### Options

```
  -a, --account-number uint      The account number of the signing account (offline mode only)
  -b, --broadcast-mode string    Transaction broadcasting mode (sync|async|block) (default "sync")
      --dry-run                  ignore the --gas flag and perform a simulation of a transaction, but don't broadcast it
      --fee-account string       Fee account pays fees for the transaction instead of deducting from the signer
      --fees string              Fees to pay along with transaction; eg: 10uatom
      --from string              Name or address of private key with which to sign
      --gas string               gas limit to set per-transaction; set to "auto" to calculate sufficient gas automatically (default 200000)
      --gas-adjustment float     adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)
      --gas-prices string        Gas prices in decimal format to determine the transaction fee (e.g. 0.1uatom)
      --generate-only            Build an unsigned transaction and write it to STDOUT (when enabled, the local Keybase is not accessible)
  -h, --help                     help for farming-adjust-reward
      --keyring-backend string   Select keyring's backend (os|file|kwallet|pass|test|memory) (default "os")
      --keyring-dir string       The client Keyring directory; if omitted, the default 'home' directory will be used
      --ledger                   Use a connected Ledger device
      --node string              <host>:<port> to tendermint rpc interface for this chain (default "tcp://localhost:26657")
      --note string              Note to add a description to the transaction (previously --memo)
      --offline                  Offline mode (does not allow any online functionality
  -o, --output string            Output format (text|json) (default "json")
  -s, --sequence uint            The sequence number of the signing account (offline mode only)
      --sign-mode string         Choose sign mode (direct|amino-json), this is an advanced feature
      --timeout-height uint      Set a block timeout height to prevent the tx from being committed past a certain height
  -y, --yes                      Skip tx broadcasting prompt confirmation
```

### Options inherited from parent commands

```
      --chain-id string   The network chain ID
```

### SEE ALSO

* [sgnd tx gov submit-proposal](sgnd_tx_gov_submit-proposal.md)	 - Submit a proposal along with an initial deposit

###### Auto generated by spf13/cobra
