# Message Executor

The message executor queries SGN for messages to be executed on-chain and submits them. A detailed walkthrough guide can be found in the [Integration Guide](https://im-docs.celer.network/developer/integration-guide#executor)'s "Executor" section

## Steps to Run Executor

### 1. Prepare the Executor Home Directory

Home directory structure

```sh
executor/
  - config/
      - executor.toml
      - cbridge.toml
  - eth-ks/
      - signer.json
```

```sh
# clone this repo first
git clone https://github.com/celer-network/sgn-v2-networks.git

# create a home directory for executor
mkdir ~/.executor

# copy this directory to the executor home
cd executor
cp ./ ~/.executor/

# copy your signer keystore file to eth-ks/
cp <your-keystore-file-path> ~/.executor/eth-ks

# extract the executor binary
cp ../binaries/<latest-executor-version>.tar.gz ~/.executor/
cd ~/.executor
tar -xz <latest-executor-version>.tar.gz
cp <latest-executor-version> ~/usr/local/bin
```

### 2. Setup Executor's Database

In theory, executor supports any databases that supports any postgresql dialect database, but it's only tested with CockroachDB for now

The following is a script for installing CockroachDB. Or you can visit their [website](https://www.cockroachlabs.com/docs/v21.2/install-cockroachdb-linux) for more detailed instruction

```sh
# install CockroachDB
curl https://binaries.cockroachdb.com/cockroach-v21.2.4.linux-amd64.tgz | tar -xz && sudo cp -i cockroach-v21.2.4.linux-amd64/cockroach /usr/local/bin/
mkdir -p /usr/local/lib/cockroach
cp -i cockroach-v21.2.4.linux-amd64/lib/libgeos.so /usr/local/lib/cockroach/
cp -i cockroach-v21.2.4.linux-amd64/lib/libgeos_c.so /usr/local/lib/cockroach/
```

```sh
# create the data directory
mkdir ~/.crdb-node0
# start a single node DB instance
cockroach start-single-node --store=~/.crdb-node0 --listen-addr=localhost:26257 --http-addr=localhost:38080 --background --insecure
# note: DB schemas will be auto added when executor starts
```

### 3. Deploy the Executor Service

You probably want to run executor using systemd so it can start with the system and restart when crash happens.

```sh
# create an executor.service file in /etc/systemd/system
touch /etc/systemd/system/executor.service
# create the log directory for executor
mkdir -p /var/log/executor
```

IMPORTANT: check if the user, user group and paths defined in your systemd file are correct

```
# executor.service

[Unit]
Description=Executor pulls active messages from sgn and executes them on-chain
After=network-online.target

[Service]
ExecStart=/usr/local/bin/executor start --loglevel debug --home <your-executor-home>
StandardOutput=append:/var/log/executor/app.log
StandardError=append:/var/log/executor/app.log
Restart=always
RestartSec=10
User=ubuntu
Group=ubuntu
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

Enable and start executor service

```sh
sudo systemctl enable executor
sudo systemctl start executor
```

Check if logs look ok

```sh
tail -f -n 200 /var/log/executor/app.log
```

## Configurations

### executor.toml

A complete executor.toml looks like this. Individual configs will be broken down and explained later.

```toml
[executor]
enable_auto_refund = false
[executor.retry]
initial_backoff_seconds = 30 # default 30
interval_seconds = 30 # default 30
scaling_factor = 1 # default 1

[server] # optional
enable = true # default false
port = 24242

[[service]] # a service represents a set of contracts and an account
id = "my-service" # optional if only one service is configured
signer_keystore = "<path-to-signer-keystore-json>"
signer_passphrase = "<keystore-passphrase>"
[[service.contracts]]
chain_id = 1
address = "<app-contract-address>" # this contract address is used to query messages from SGN
allow_sender_groups = ["my-contracts"] # mount sender groups here. if not configured, sender checking is skipped
[[service.contracts]]
chain_id = 56
address = "<app-contract-address>"
allow_sender_groups = ["my-contracts"] # you can also list multiple sender groups. message execution is allowed if a sender is found in ANY of the sender groups
[[service.contract_sender_groups]]
name = "my-contracts"
allow = [
  { chain_id = 1, address = "<app-contract-address>" },
  { chain_id = 56, address = "<app-contract-address>" },
]

[sgnd]
sgn_grpc = "cbridge-prod2.celer.network:9094"
gateway_grpc = "cbridge-prod2.celer.network:9094"

[db]
url = "localhost:26257"

[alert]
type = "slack" # only slack is supported for now
webhook = "<slack-web-hook-url>"
[[alert.low_gas_thresholds]]
chain_id = 1
threshold = "2000000000000000000"
[[alert.low_gas_thresholds]]
chain_id = 56
threshold = "1000000000000000000"
```

### Auto Refund

For messages that has attached transfers, sometimes the transfer would fail at bridge due to slippage or other reasons. You can enable executor's auto refund config to let executor handle the refund case for you. If this is enabled, executor automatically calls the MessageBus's `executeMessageWithTransferRefund()` on source chain.

```toml
enable_auto_refund = true # if not configured, the default is false
```

### Retry Strategies

The execution of a message can fail due to various errors such as connection issues and rpc provider nodes inconsistency. The executor tries to re-submit errored executions according to this setting (Note that messages in the "failed" status do not get re-submitted).

| key                     | default | description                                                                                   |
| ----------------------- | ------- | --------------------------------------------------------------------------------------------- |
| initial_backoff_seconds | 30      | The amount of seconds to wait before retrying for the first time                              |
| interval_seconds        | 30      | The amount of seconds to wait in-between retries                                              |
| scaling_factor          | 1       | For any attempt# n, x = scaling factor, s = interval seconds, actual retry interval = s * n^x |

### Service

From now on, multi-client is supported for one executor instance. That means for every msg, you could decide which signer would be used to execute it. You can configue different clients by simply duplicating `[[service]]` within `executor.toml`. For each `[[service]]`, `signer_keystore` and `signer_passphrase` define the signer used by this `[[service]]` to send tx. `[[service.contracts]]` defines which contracts on which chains are cared by this `[[service]]`. A `[[service]]` would only execute msgs that went to an already configurred pair of contract addr and chain id. Besides, [Sender Groups](#sender-groups) is another available config option for `[[service]]`.

### Sender Groups

Under the current IM architecture, any contracts can send messages to any other contracts. This means that a malicious party can forge messages that conform to your contracts' message data type, send it to your contract, and exhaust your executor's gas fund. Thus, in production, it is important that executor checks where a message is originated from. Sender groups are designed just for that.

If you have experience with cloud services such as AWS, your might recognize that a sender group is pretty much a "security group".

An example sender group looks like this

```toml
[[service.contract_sender_groups]]
# the name/ID of the group. service.contracts refer to a sender group in allow_sender_groups
name = "my-contracts" 
allow = [
  # allow and execute messages originated from <app-contract-address> on chain 1
  { chain_id = 1, address = "<app-contract-address>" },
  # allow and execute messages originated from <app-contract-address> on chain 56
  { chain_id = 56, address = "<app-contract-address>" },
]
```

### Low Gas Alert

In production, you might want to monitor your executor's balance of gas token on each chain. In this config you can specify a threshold amount for each chain. Executor periodically checks the balance and once the threshold is hit, it triggers an alert action. Currently only slack alert is supported.

```toml
[alert]
type = "slack" # only slack is supported for now
webhook = "<slack-web-hook-url>"
[[alert.low_gas_thresholds]]
chain_id = 1
threshold = "2000000000000000000" # balance in wei
[[alert.low_gas_thresholds]]
chain_id = 56
threshold = "1000000000000000000"
```

## Status & Meanings

### Unknown (0)

placeholder status that should never happen

### Unexecuted (1)

when a message is queried from SGN, it is first saved as this status

### Init_Refund_Executing (2)

only applicable if "enable_auto_refund" is on. indicates that the init refund call to SGN Gateway is ongoing.

### Init_Refund_Executed (3)

only applicable if "enable_auto_refund" is on. indicates the init refund call is finished and executor has got the necessary data to submit the refund withdrawal on-chain.

### Init_Refund_Failed (4)

only applicable if "enable_auto_refund" is on. indicates that the executor failed to call SGN Gateway to acquire the refund data.

### Executing (5)

transient status. indicates that the executor is attempting to execute the message.

### Succeeded (6)

final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the message/messageWithTransfer/refund is executed successfully on-chain.

### Fallback (7)

final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the submited tx has a successful status but a revert happens in your application contract. MessageBus called your contract's executeMessageWithTransferFallback() function.

### Failed (8)

final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the tx calling MessageBus has failed OR the call from MessageBus to your contract's executeMessageWithTransferFallback() function has failed.

### Ignored (9)

final status. only applicable if you have executor.contracts.allow_sender_groups and executor.contract_sender_groups configured. indicates that a message sent to one of your contracts is not originated from the allowed sender you defined.

## Commands

### Action Commands

Actions commands are a means of interacting with the executor DB and on-chain data to fix various issues.

```sh
executor tx unstuck --id <message-id> --submitter <optional, service-id>
```

This command does the following things:

1. looks up a message by its id
2. submits an empty tx onchain at the current pending nonce of the executor account
3. reverts the message to UNEXECUTED, so that it will get executed in the next around

Note: the `--submitter` flag is optional if there is only one `[[service]]` configured. if there are multiple `[[service]]`, you need to identify which submitter keystore you want to use by specifying the configured `[[service.id]]`

There are several optional flags to set, check use `--help` to see your options.

### Query Commands

```sh
executor query message <message-id>
```

Queries a message from both local DB and SGN

### REST API Server

Executor can optionally expose a server at a configured port. The server is a wrapper around the aforementioned "commands".

```toml
# executor.toml
[server] 
enable = true # default false
port = 24242
```

#### POST /get-message

```json
{
    // identifier, use one of the following three
    "id": "<message-id>",
    "srcTx": "<source-tx-hash>",
    "dstTxHash": "<destination-tx-hash>",
}
```

#### POST /unstuck

```json
// Request Body
{
    // identifier, use one of the following three
    "id": "<message-id>",
    "srcTx": "<source-tx-hash>",
    "dstTxHash": "<destination-tx-hash>",

    // submitter, omit if there is only one service configured
    "submitter": "<service-id>",

    // tx options, use either gasPrice or (gasTipCap and gasFeeCap)
    // legacy tx
    "gasPrice": "<gas-price>",
    // eip1559 tx
    "gasTipCap": "<gas-tip-cap>",
    "gasFeeCap": "<gas-fee-cap>"
}
```
