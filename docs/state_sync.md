# State Sync Manual

State sync is the fastest way to bring a new SGN node up to the current block height or rescue one
that failed to join consensus for some reason. For more information, read this
[blog post](https://blog.cosmos.network/cosmos-sdk-state-sync-guide-99e4cf43be2f).

## Prepare the nodes providing the state snapshot

NOTE: These steps have already been performed on the Celer foundation nodes, so snapshots should be readily available.

1. Check the Tendermint config file `$HOME/.sgnd/config/config.toml`:

    Make sure port 26657 is accessible externally:

    ```toml
    [rpc]
    laddr = "tcp://0.0.0.0:26657"
    ```

2. Check the Cosmos SDK config file `$HOME/.sgnd/config/app.toml`:

    ```toml
    [state-sync]
    snapshot-interval = 1000 # Use 100 in case of emergency, must be a multiple of 100 or pruning-keep-every if set
    snapshot-keep-recent = 2
    ```

3. Restart the node with the correct configs if necessary.

4. Repeat for another node. State sync requires fetching snapshots from **at least two** up-to-date
nodes.

## Prepare the node receiving the state snapshot

1. Install `jq` if necessary.

2. Query for a trusted block height and hash from an up-to-date node. First, query for the most recent block:

    ```sh
    curl -s http://<up-to-date-node-ip>:26657/block | \
      jq -r '.result.block.header.height
    ```

    The command should print a block height and hash like `2077077`.

    Given that Celer foundation nodes take a snapshot every 1000 blocks, to speedup state-sync, you
    can set the trusted block height to the last multiple of 1000. Query for the block hash at that
    block:

    ```sh
    curl -s http://<up-to-date-node-ip>:26657/block?height=<latest-snapshot-height> | \
      jq -r '.result.block.header.height + "\n" + .result.block_id.hash'
    ```

    The command should print a block height and hash like:

    ```
    2077000
    6FD28DAAAC79B77F589AE692B6CD403412CE27D0D2629E81951607B297696E5B
    ```

3. Modify the Tendermint config file `$HOME/.sgnd/config/config.toml`:

    ```toml
    [statesync]
    enable = true
    rpc_servers = "<up-to-date-node-A-ip>:26657,<up-to-date-node-B-ip>:26657"
    trust_height = 2077000 # <trusted-block-height>
    trust_hash = "6FD28DAAAC79B77F589AE692B6CD403412CE27D0D2629E81951607B297696E5B" # <trusted-block-hash>
    trust_period = "6h" # Recommended to be about 2/3 of unbonding time
    ```

## Syncing a new node

```sh
sudo systemctl start sgnd
```

## Re-syncing a node with corrupt local state

1. Stop the node and clear local data:

    ```sh
    sudo systemctl stop sgnd # Stop if running
    sgnd unsafe-reset-all
    ```

2. Make sure `$HOME/.sgnd/data/priv_validator_state.json` is reset to the initial state of:

    ```json
    {
      "height": "0",
      "round": 0,
      "step": 0
    }
    ```

    and the `$HOME/.sgnd/data/snapshots` folder exists.

3. Restart the node:

    ```sh
    sudo systemctl start sgnd
    ```

    Monitor `tendermint.log`:

    ```sh
    tail -f /var/log/sgnd/tendermint.log
    ```

    If everything is setup correctly, you should see the node waits for a state snapshot from the peers
    and syncs itself once a snapshot is available. You can tell that a state sync is successful once the
    log starts printing new blocks.
