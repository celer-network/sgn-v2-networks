# Message Executor

The message executor queries SGN for messages to be executed on-chain and submits them. A detailed walkthrough guide can be found in the [Integration Guide](https://im-docs.celer.network/developer/integration-guide#executor)'s "Executor" section

## Status & Meanings

### Unknown
placeholder status that should never happen

### Unexecuted
when a message is queried from SGN, it is first saved as this status

### Init_Refund_Executing
only applicable if "enable_auto_refund" is on. indicates that the init refund call to SGN Gateway is ongoing.

### Init_Refund_Executed
only applicable if "enable_auto_refund" is on. indicates the init refund call is finished and executor has got the necessary data to submit the refund withdrawal on-chain.

### Init_Refund_Failed
only applicable if "enable_auto_refund" is on. indicates that the executor failed to call SGN Gateway to acquire the refund data.

### Executing
transient status. indicates that the executor is attempting to execute the message.

### Succeeded
final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the message/messageWithTransfer/refund is executed successfully on-chain.

### Fallback
final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the submited tx has a successful status but a revert happens in your application contract. MessageBus called your contract's executeMessageWithTransferFallback() function.

### Failed
final status. changed when the executor receives the "Executed" event from the MessageBus contract. indicates that the tx calling MessageBus has failed OR the call from MessageBus to your contract's executeMessageWithTransferFallback() function has failed.

### Ignored
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
