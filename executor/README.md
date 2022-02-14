# Message Executor

The message executor queries SGN for messages to be executed on-chain and submits them

Current release doesn't support re-submitting if the execution fails due to

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
cp <your-keystore-file> ~/.executor/eth-ks

# extract the executor binary
cp ../binaries/executor-v1.6.0.-dev.1-linux-amd64.tar.gz ~/.executor/
cd ~/.executor
tar -xz executor-v1.6.0-dev.1-linux-amd64.tar.gz
cp executor-v1.6.0-dev.1-linux-amd64 ~/usr/local/bin
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
ExecStart=/usr/local/bin/executor -loglevel=debug -home=<your-executor-home>
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
