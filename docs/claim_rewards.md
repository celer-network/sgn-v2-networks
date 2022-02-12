# Claim Reward Manual

This manual describes the process of claiming reward, including cbridge fee, pegbridge fee and staking reward.

**NOTE: The manual assumes familiarity with Unix command line and blockchain validator nodes.
Prior experience with Cosmos SDK and Ethereum transactions will be helpful.**

## Table of Contents
- [Prerequisite](#prerequisite)
- [Claim cbridge fee](#claim-cbridge-fee)
- [Claim pegbridge fee](#claim-pegbridge-fee)
- [Claim staking reward](#claim-staking-reward)

## Prerequisite

Before claiming any reward, make sure you have read and followed the [Validator Manual](validator.md) for every operation.

## Claim cbridge fee

Query all claimable cbridge fee, as well as token amount and token value in usd:
```shell
sgnd query cbridge fee-share <validator-eth-address>
```

Along with a `wdlist` flag, we can generate a withdrawal list which will be used later.
```shell
sgnd query cbridge fee-share <validator-eth-address> --wdlist

```
The detail of withdrawal list locates at the bottom of above command output, after a line like `validator withdraw fee inputs:`. In fact, each line within withdrawal list represents a withdrawal request of one kind of token, and will require one transaction to accomplish that withdrawal. It's probable for some of those withdrawals that its value does not cover a transaction gas cost. So we highly recommend using `min-usd` flag when generating withdraw list. A `min-usd` of `100` would filter out any withdrawal request whose token value is less than 100 usd. Just like this:
```shell
sgnd query cbridge fee-share <validator-eth-address> --wdlist --min-usd 100
```

Once you decided your favorite withdrawal list, create a new file and save it. Then we're ready to claim cbridge fee.
```shell
touch claim_fee.txt
// use your favorite text editor to write the withdrawal list in and save it.
// submit a sgn transaction
echo $COSMOS_KEYRING_PASSPHRASE | sgnd tx cbridge validator-claim-fee --file claim_fee.txt
// check withdrawal request status
echo $COSMOS_KEYRING_PASSPHRASE | sgnd ops validator withdraw-cbr-fee --file claim_fee.txt --query
// waiting for status of all requests come to [WD_WAITING_FOR_LP]
echo $COSMOS_KEYRING_PASSPHRASE | sgnd ops validator withdraw-cbr-fee --file claim_fee.txt
```

If none obvious error happens, congratulations, we've done for claiming cbridge fee.


## Claim pegbridge fee

The process of claiming pegbridge fee is very similar to claiming cbridge fee.

Query all claimable cbridge fee, as well as token amount and token value in usd:
```shell
sgnd query pegbridge fee <validator-eth-address>
```

Along with a `wdlist` flag, we can generate a withdrawal list which will be used later.
```shell
sgnd query pegbridge fee <validator-eth-address> --wdlist

```
The detail of withdrawal list locates at the bottom of above command output, after a line like `validator withdraw fee inputs:`. In fact, each line within withdrawal list represents a withdrawal request of one kind of token, and will require one transaction to accomplish that withdrawal. It's probable for some of those withdrawals that its value does not cover a transaction gas cost. So we highly recommend using `min-usd` flag when generating withdraw list. A `min-usd` of `100` would filter out any withdrawal request whose token value is less than 100 usd. Just like this:
```shell
sgnd query pegbridge fee <validator-eth-address> --wdlist --min-usd 100
```

Once you decided your favorite withdrawal list, create a new file and save it. Then we're ready to claim cbridge fee.
```shell
touch claim_fee.txt
// use your favorite text editor to write the withdrawal list in and save it.
// submit a sgn transaction
echo $COSMOS_KEYRING_PASSPHRASE | sgnd tx pegbridge validator-claim-fee --file claim_fee.txt
// check withdrawal request status
echo $COSMOS_KEYRING_PASSPHRASE | sgnd ops validator withdraw-pegbr-fee --file claim_fee.txt --query
// waiting for status of all requests come to [enough signatures]
echo $COSMOS_KEYRING_PASSPHRASE | sgnd ops validator withdraw-pegbr-fee --file claim_fee.txt
```

If none obvious error happens, congratulations, we've done for claiming pegbridge fee.

## Claim staking reward

Query claimable staking reward in CELR:
```shell
sgnd query distribution staking-reward-info <validator-eth-address>
```

Submit a sgn transaction for claiming your accumulative staking reward:
```shell
echo $COSMOS_KEYRING_PASSPHRASE | sgnd tx distribution claim-staking-reward <validator-eth-address>
```

Check claiming request status:
```shell
sgnd ops validator claim-staking-reward --query
```

Until your request status comes to `enough signatures`, then:
```shell
sgnd ops validator claim-staking-reward
```

If none obvious error happens, congratulations, we've done for claiming staking reward.
