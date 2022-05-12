# Message Executor

The message executor queries SGN for messages to be executed on-chain and submits them. A detailed walkthrough guide can be found in the [Integration Guide](https://im-docs.celer.network/developer/integration-guide#executor)'s "Executor" section

## Configurations

### executor.toml

A complete executor.toml looks like this. Individual configs will be broken down and explained later.

```toml
 [executor]
 enable_auto_refund = false
 
 [[service]] # executor supports multi-client service. every service instance represents one client with different configuration.
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

#### Unknown (0)

placeholder status that should never happen

#### Unexecuted (1)

when a message is queried from SGN, it is first saved as this status

#### Init_Refund_Executing (2)

only applicable if "enable_auto_refund" is on. indicates that the init refund call to SGN Gateway is ongoing.

#### Init_Refund_Executed (3)

only applicable if "enable_auto_refund" is on. indicates the init refund call is finished and executor has got the necessary data to submit the refund withdrawal on-chain.

#### Init_Refund_Failed (4)

only applicable if "enable_auto_refund" is on. indicates that the executor failed to call SGN Gateway to acquire the refund data.

#### Executing (5)

transient status. indicates that the executor is attempting to execute the message.

#### Succeeded (6)

final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the message/messageWithTransfer/refund is executed successfully on-chain.

#### Fallback (7)

final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the submited tx has a successful status but a revert happens in your application contract. MessageBus called your contract's executeMessageWithTransferFallback() function.

#### Failed (8)

final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the tx calling MessageBus has failed OR the call from MessageBus to your contract's executeMessageWithTransferFallback() function has failed.

#### Ignored (9)

final status. only applicable if you have executor.contracts.allow_sender_groups and executor.contract_sender_groups configured. indicates that a message sent to one of your contracts is not originated from the allowed sender you defined.

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

## Action Commands

Actions commands are a means of interacting with the executor DB and on-chain data to fix various issues.

```sh
executor tx unstuck --id <message-id>
```

This command does the following things: 
1. looks up a message by its id 
2. submits an empty tx onchain at the current pending nonce of the executor account
3. reverts the message to UNEXECUTED, so that it will get executed in the next around

There are several optional flags to set, check use `--help` to see your options.

## Query Commands

You can query the status of a message by command below with its id:

```sh
executor query message <message-id>
```

The query logic is very simple. There are two steps to find the message we need.

1. query the message in local-db by its id.
2. if no message found in local-db, then query the message in sgn by its id.

Once we found the right message, its status would be printed out directly. You can take a signt at [Status & Meanings](#status--meanings) to get an explanation of the status.

If you got an error like "failed to load (whatever-path)/executor.toml", please re-run the query command with `home` flag just like this:

```sh
executor query message <message-id> --home <your-executor-home>
```

## Process remote query request

If you want to remotely interact with your executor service to query a message's status, restart your executor service with `server` flag. Replace `ExecStart` in `executor.service` file by:

```sh
ExecStart=/usr/local/bin/executor start --loglevel debug --home <your-executor-home> --server
```

Then restart your service

```sh
sudo systemctl restart executor
```

That would start a local http server listening on 8090 by default. Port of this server can be modified by `port` flag. Like:

```sh
ExecStart=/usr/local/bin/executor start --loglevel debug --home <your-executor-home> --server --port <your-preferred-port>
```

Once you restart your executor with `server` flag, you can send a query request about a message's status to your executor by:

```sh
curl http://localhost:<port>/message-status?messageId=<message-id>
```

Or just go to `http://localhost:<port>/message-status?messageId=<message-id>` on your explorer. Then you'll get your response.
