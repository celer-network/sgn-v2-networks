# Message Executor

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
# create a home directory for executor
mkdir ~/.executor
# copy this directory to the executor home
cp ./ ~/.executor/
# copy your signer keystore file to eth-ks/
cp <your-keystore-file> ~/.executor/eth-ks
# extract the executor binary
cd ~/.executor
tar -xz executor-v1.6.0-dev-linux-amd64.tar.gz
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
# schemas will be auto added when executor starts
```

### 3. Deploy the Executor Service

You probably want to run executor using systemd so it can start with the system and restart when crash happens.

```sh
# create a executor.service file in /etc/systemd/system
touch /etc/systemd/system/executor.service
```

```
# executor.service
[Unit]
Description=Executor pulls active messages from sgn and executes them on-chain
After=network-online.target

[Service]
ExecStart=/home/ubuntu/go/bin/executor -loglevel=debug -home=/home/ubuntu/.executor
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

Before starting the systemd service, don't forget to create the log directory first

```sh
mkdir -p /var/log/executor
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
