# Upgrade Manual

The SGN chain can be upgraded to new software versions via on-chain governance. The general procedure is as follows:

1. In the communication channel, validator operators agree on the post-upgrade binary version and the upgrade height.

2. Validator operators download and prepare the new binary.

3. Validator operators need to make sure `sgnd` is running smoothly on the same **pre-upgrade** binary version. Use `sgnd version` to verify the binary version.

4. The upgrade coordinator submits the software upgrade proposal:

```sh
sgnd tx gov submit-proposal software-upgrade <upgrade-name> --title <upgrade-name> --description <upgrade-name> --upgrade-height <target-height> --home ~/.sgnd
```

5. The coordinator shares the proposal ID in the communication channel.

6. Validator operators vote on the proposal:

```sh
echo {validator_sgn_passphrase} | sgnd tx gov vote {proposal_id} yes --home ~/.sgnd
```

7. Validator operators monitor `tendermint.log`, wait for `sgnd` to reach the target height and halt automatically.

8. Validator operators restart `sgnd` with the **post-upgrade** binary and monitor `tendermint.log`. Confirm that new blocks are being produced.
