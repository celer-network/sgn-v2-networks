## Governance Manual

This manual describes the processes to submit a gov proposal and vote a proposal.

As of the writing of this manual, the gov params are set as:

```json
{
  "voting_params":{
    "voting_period":"300000000000"
  },
  "tally_params":{
    "quorum":"0.667000000000000000",
    "threshold":"0.667000000000000000",
    "veto":"0.334000000000000000"
  },
  "deposit_params":{
    "min_deposit":"0",
    "max_deposit_period":"120000000000"
  }
}
```

Refer to [Query the parameters of the governance](./sgnd/sgnd_query_gov_params.md) for current settings.

- `voting_period` indicates how long a proposal is waiting for vote, in nanoseconds
- tally `quorum` indicates how much percent of the voting power should be achieved to vote the proposal, if there is not enough params.quorum of votes, the proposal fails
- tally `veto`, if more than params.veto of voters veto, proposal fails
- tally `threshold`, if more than params.threshold of non-abstaining voters vote Yes, proposal passes
- `deposit_params`, current system accepts 0 deposit to propose a proposal

Note that to update these params themselves, a param change governance process is needed.

We usually propose and vote on two types of governance proposals, 1. simple param changes for a certain module and 2. cbridge / pegbridge config updates. There are
other types of governance proposals such as farming adjustments and software upgrades, but they happen less often.

### Param Change

1. Query current staking syncer duration and submit change proposal:

NOTE: Replace the placeholder {proposal_id} with the real proposal ID in the following commands. You can find the proposal ID in the output after submitting a proposal.

```sh
sgnd query staking params --home ~/.sgnd
sgnd tx gov submit-proposal param-change ./gov-example/param_change_proposal.json --home ~/.sgnd
sgnd query gov proposal {proposal_id} --home ~/.sgnd
```

2. Perform the command below to vote yes and then wait for enough voters to vote yes in `voting_period`:

```sh
echo {validator_sgn_passphrase} | sgnd tx gov vote {proposal_id} yes --home ~/.sgnd
```

3. Query proposal status:

```sh
sgnd query gov proposal {proposal_id} --home ~/.sgnd
sgnd query staking params --home ~/.sgnd
```

Edit the subspace and key accordingly in `./gov-example/param_change_proposal.json` for updating the other params in other modules.

### Cbridge Config Update

cbridge / pegbridge config updates are used to add new chains and tokens.

1. Query current cbr config and submit change proposal:

```sh
sgnd query cbridge config --home ~/.sgnd
sgnd tx gov submit-proposal cbridge-change ./gov-example/cbridge_cbr_proposal.json --home ~/.sgnd
sgnd query gov proposal {proposal_id} --home ~/.sgnd
```

2. Perform the command below to vote yes and then wait for enough voters to vote yes in `voting_period`:

```sh
echo {validator_sgn_passphrase} | sgnd tx gov vote {proposal_id} yes --home ~/.sgnd
```

3. Query proposal status:

```sh
sgnd query gov proposal {proposal_id} --home ~/.sgnd
sgnd query cbridge config --home ~/.sgnd
```
### SEE ALSO

* [sgnd tx gov](sgnd_tx_gov.md)	 - Governance transactions subcommands
* [sgnd tx gov submit-proposal cbridge-change](sgnd_tx_gov_submit-proposal_cbridge-change.md)	 - Submit a cbridge config change proposal
* [sgnd tx gov submit-proposal farming-add-pool](sgnd_tx_gov_submit-proposal_farming-add-pool.md)	 - Submit an AddPoolProposal
* [sgnd tx gov submit-proposal farming-add-tokens](sgnd_tx_gov_submit-proposal_farming-add-tokens.md)	 - Submit an AddTokensProposal
* [sgnd tx gov submit-proposal farming-adjust-reward](sgnd_tx_gov_submit-proposal_farming-adjust-reward.md)	 - Submit an AdjustRewardProposal
* [sgnd tx gov submit-proposal farming-set-reward-contracts](sgnd_tx_gov_submit-proposal_farming-set-reward-contracts.md)	 - Submit an SetRewardContractsProposal
* [sgnd tx gov submit-proposal mint-adjust-provisions](sgnd_tx_gov_submit-proposal_mint-adjust-provisions.md)	 - Submit an AdjustProvisionsProposal
* [sgnd tx gov submit-proposal param-change](sgnd_tx_gov_submit-proposal_param-change.md)	 - Submit a parameter change proposal
* [sgnd tx gov submit-proposal pegbridge-change](sgnd_tx_gov_submit-proposal_pegbridge-change.md)	 - Submit a pegbridge config change proposal
* [sgnd tx gov submit-proposal pegged-pair-delete](sgnd_tx_gov_submit-proposal_pegged-pair-delete.md)	 - Submit a pegged pair delete proposal
* [sgnd tx gov submit-proposal software-upgrade](sgnd_tx_gov_submit-proposal_software-upgrade.md)	 - Submit a software upgrade proposal
