## sgnd tx gov submit-proposal

Submit a proposal along with an initial deposit

### Synopsis

Submit a proposal along with an initial deposit.
Proposal title, description, type and deposit can be given directly or through a proposal JSON file.

Example:
$ <appd> tx gov submit-proposal --proposal="path/to/proposal.json" --from mykey

Where proposal.json contains:

{
  "title": "Test Proposal",
  "description": "My awesome proposal",
  "type": "Text",
  "deposit": "10test"
}

Which is equivalent to:

$ <appd> tx gov submit-proposal --title="Test Proposal" --description="My awesome proposal" --type="Text" --deposit="10test" --from mykey

```
sgnd tx gov submit-proposal [flags]
```

### Options

```
  -a, --account-number uint      The account number of the signing account (offline mode only)
  -b, --broadcast-mode string    Transaction broadcasting mode (sync|async|block) (default "sync")
      --deposit string           The proposal deposit
      --description string       The proposal description
      --dry-run                  ignore the --gas flag and perform a simulation of a transaction, but don't broadcast it
      --fee-account string       Fee account pays fees for the transaction instead of deducting from the signer
      --fees string              Fees to pay along with transaction; eg: 10uatom
      --from string              Name or address of private key with which to sign
      --gas string               gas limit to set per-transaction; set to "auto" to calculate sufficient gas automatically (default 200000)
      --gas-adjustment float     adjustment factor to be multiplied against the estimate returned by the tx simulation; if the gas limit is set manually this flag is ignored  (default 1)
      --gas-prices string        Gas prices in decimal format to determine the transaction fee (e.g. 0.1uatom)
      --generate-only            Build an unsigned transaction and write it to STDOUT (when enabled, the local Keybase is not accessible)
  -h, --help                     help for submit-proposal
      --keyring-backend string   Select keyring's backend (os|file|kwallet|pass|test|memory) (default "os")
      --keyring-dir string       The client Keyring directory; if omitted, the default 'home' directory will be used
      --ledger                   Use a connected Ledger device
      --node string              <host>:<port> to tendermint rpc interface for this chain (default "tcp://localhost:26657")
      --note string              Note to add a description to the transaction (previously --memo)
      --offline                  Offline mode (does not allow any online functionality
  -o, --output string            Output format (text|json) (default "json")
      --proposal string          Proposal file path (if this path is given, other proposal flags are ignored)
  -s, --sequence uint            The sequence number of the signing account (offline mode only)
      --sign-mode string         Choose sign mode (direct|amino-json), this is an advanced feature
      --timeout-height uint      Set a block timeout height to prevent the tx from being committed past a certain height
      --title string             The proposal title
      --type string              The proposal Type
  -y, --yes                      Skip tx broadcasting prompt confirmation
```

### Options inherited from parent commands

```
      --chain-id string   The network chain ID
```

### SEE ALSO

* [sgnd tx gov](sgnd_tx_gov.md)	 - Governance transactions subcommands
* [sgnd tx gov submit-proposal cbridge-change](sgnd_tx_gov_submit-proposal_cbridge-change.md)	 - Submit a cbridge config change proposal
* [sgnd tx gov submit-proposal farming-add-pool](sgnd_tx_gov_submit-proposal_farming-add-pool.md)	 - Submit an AddPoolProposal
* [sgnd tx gov submit-proposal farming-add-tokens](sgnd_tx_gov_submit-proposal_farming-add-tokens.md)	 - Submit an AddTokensProposal
* [sgnd tx gov submit-proposal farming-adjust-reward](sgnd_tx_gov_submit-proposal_farming-adjust-reward.md)	 - Submit an AdjustRewardProposal
* [sgnd tx gov submit-proposal farming-batch-add-pool](sgnd_tx_gov_submit-proposal_farming-batch-add-pool.md)	 - Submit a BatchAddPoolProposal
* [sgnd tx gov submit-proposal farming-batch-adjust-reward](sgnd_tx_gov_submit-proposal_farming-batch-adjust-reward.md)	 - Submit an batchAdjustRewardProposal
* [sgnd tx gov submit-proposal farming-set-reward-contracts](sgnd_tx_gov_submit-proposal_farming-set-reward-contracts.md)	 - Submit an SetRewardContractsProposal
* [sgnd tx gov submit-proposal message-update](sgnd_tx_gov_submit-proposal_message-update.md)	 - Submit a message buses update proposal
* [sgnd tx gov submit-proposal mint-adjust-provisions](sgnd_tx_gov_submit-proposal_mint-adjust-provisions.md)	 - Submit an AdjustProvisionsProposal
* [sgnd tx gov submit-proposal param-change](sgnd_tx_gov_submit-proposal_param-change.md)	 - Submit a parameter change proposal
* [sgnd tx gov submit-proposal pegbridge-change](sgnd_tx_gov_submit-proposal_pegbridge-change.md)	 - Submit a pegbridge config change proposal
* [sgnd tx gov submit-proposal pegged-pair-delete](sgnd_tx_gov_submit-proposal_pegged-pair-delete.md)	 - Submit a pegged pair delete proposal
* [sgnd tx gov submit-proposal software-upgrade](sgnd_tx_gov_submit-proposal_software-upgrade.md)	 - Submit a software upgrade proposal
* [sgnd tx gov submit-proposal total-supply-update](sgnd_tx_gov_submit-proposal_total-supply-update.md)	 - Submit a pegged total supply update proposal

###### Auto generated by spf13/cobra
