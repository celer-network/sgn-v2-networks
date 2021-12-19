# Validator Manual

This manual describes the process of spinning up an SGN node and joining the network as a validator.

**NOTE: The manual assumes familiarity with Unix command line and blockchain validator nodes.
Prior experience with Cosmos SDK and Ethereum transactions will be helpful.**

## Table of Contents
- [Prepare machine and install dependencies](#prepare-machine-and-install-dependencies)
- [Setup binary, config and accounts](#setup-binary-config-and-accounts)
- [Run node with systemd](#run-node-with-systemd)
- [Claim validator status](#claim-validator-status)

If you only intend to run an SGN node to sync and verify the blocks, stop after [run node with systemd](#run-validator-with-systemd).

## Prepare machine and install dependencies

We run on Ubuntu Linux amd64 with Amazon EC2 as an example. Feel free to experiment with other VPS or physical server setups on your own.

1. Start an EC2 machine with the Ubuntu 20.04 LTS image. We recommend using `t3.medium` with an EBS volume of at least
100GB for testnet and `c5a.2xlarge` with an EBS volume of at least 500GB for mainnet. Use the appropriate security
groups and a keypair that you have access to.

2. Install go (at least 1.16):

```sh
sudo snap install go --classic
```

Install gcc, make and libleveldb-dev

```sh
sudo apt update && sudo apt install gcc make libleveldb-dev
```

3. Set \$GOBIN and add \$GOBIN to \$PATH. Edit `$HOME/.profile` and add:

```sh
export GOBIN=$HOME/go/bin; export GOPATH=$HOME/go; export PATH=$PATH:$GOBIN
```

to the end, then:

```sh
source $HOME/.profile
```

4. (Optional) Install geth:

```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

## Setup binary, config and accounts

1. From the `/home/ubuntu` directory, download and install the `sgnd` binary:

```sh
wget <TODO: add link>
cp
```

2. From the `/home/ubuntu` directory, clone the `sgn-v2-networks` repository:

```sh
git clone https://github.com/celer-network/sgn-v2-networks
cd sgn-v2-networks/<network-name>
```

3. Copy the config files:

```sh
mkdir -p $HOME/.sgnd/config
cp * $HOME/.sgnd/config
```

4. Initialize the new validator node:

```sh
sgnd init <node-name> --chain-id <network-name> --home $HOME/.sgnd
# Overwrite genesis.json and config.toml with the ones from sgn-v2-networks
cp genesis.json config.toml $HOME/.sgnd/config
```

Backup the generated `$HOME/.sgnd/config/node_key.json` and `$HOME/.sgnd/config/priv_validator_key.json` securely. Make sure the keys are **never** committed to any repo.

3. Fill out the `moniker` field in `config/config.toml` with your `node-name`.
Fill out the `external_address` field with `<public-ip:26656>`, where the `public-ip` is the public IP of the EC2 machine to host the node. Currently, the Celer foundation nodes restrict the access to port 26656, so please **report your public IP to the Celer team** to get whitelisted.

4. Add a Cosmos SDK / Tendermint validator account:

```sh
sgnd keys add <node-name>
```

Input a passphrase for the keyring. Backup the passphrase along with the displayed mnemonic phrase securely. Make sure they are **never** committed to any repo.

To view the account created, run:

```sh
sgnd keys list
```

Make a note of the account address.

5. Prepare an Ethereum gateway URL. You can use services like [Infura](https://infura.io/),
[Alchemy](https://alchemyapi.io/) or run your own node.

6. Prepare an Ethereum keystore JSON file as the **validator key**. Assuming `geth` is installed, this can be done via:

```sh
geth account new
```

Save it as `$HOME/.sgnd/eth-ks/val.json`:

```sh
mkdir $HOME/.sgnd/eth-ks
cp <path-to-keystore-json> $HOME/.sgnd/eth-ks/val.json
```

7. Prepare another Ethereum keystore JSON as the **signer key** and save it as `$HOME/.sgnd/eth-ks/signer.json`.

8. Fill in the fields in `$HOME/.sgnd/config/sgn.toml` with the correct values:

    | Field | Description |
    | ----- | ----------- |
    | eth.gateway | The Ethereum gateway URL obtained from step 5 |
    | eth.signer_keystore | The path to the signer keystore file in step 7 |
    | eth.signer_passphrase | The passphrase of the signer keystore |
    | eth.validator_address | The Ethereum address of the validator keystore created in step 6 |
    | sgnd.passphrase | The passphrase you typed in step 4 |
    | sgnd.validator_account | The validator Cosmos SDK account added in step 4 |

9. Fill in the missing gateway URLs in `$HOME/.sgnd/config/cbridge.toml` with the corresponding JSON-RPC URLs for the chains. In general, we recommend using paid provider services instead of the public endpoints for better reliability.

## Run validator with systemd

We recommend using systemd to run your validator. Feel free to experiment with other setups on your own.

1. Prepare the sgnd system service:

```sh
sudo mkdir -p /var/log/sgnd
sudo touch /var/log/sgnd/tendermint.log
sudo touch /var/log/sgnd/app.log
sudo touch /etc/systemd/system/sgnd.service
```

Add the following to `/etc/systemd/system/sgnd.service`:

```
[Unit]
Description=SGN daemon
After=network-online.target

[Service]
Environment=HOME=/home/ubuntu
ExecStart=/home/ubuntu/go/bin/sgnd start \
  --home /home/ubuntu/.sgnd/
StandardOutput=append:/var/log/sgnd/tendermint.log
StandardError=append:/var/log/sgnd/app.log
Restart=always
RestartSec=3
User=ubuntu
Group=ubuntu
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

With this setup, `tendermint.log` contains Tendermint logs (i.e. block height, peer info, etc.) while `app.log`
contains SGN-specific logs.

4. Create `/etc/logrotate.d/sgn` and add the following:

```
/var/log/sgnd/*.log {
    compress
    copytruncate
    daily
    maxsize 30M
    rotate 30
}
```

5. Add an entry to `/etc/crontab` to make logrotate run every 6 hours:

```
30 */6  * * *   root    logrotate -f /etc/logrotate.conf
```

6. Enable and start the sgnd service:

```sh
sudo systemctl enable sgnd.service
sudo systemctl start sgnd.service
```

7.  Monitor `tendermint.log` and wait for the node to sync:

```sh
tail -f /var/log/sgnd/tendermint.log
```

You can tell the node is synced when new blocks show up about every 5 seconds.

## Claim validator status

1. For testnet, obtain some Goerli ETH from places like the Paradigm
[faucet](https://faucet.paradigm.xyz/). Contact the Celer team for some Goerli test CELR tokens and get whitelisted for a validator spot.

For mainnet, prepare real ETH and CELR tokens.

2. Send ETH and CELR to the address of the **validator key**. Make sure it has enough CELR for the intended self delegation and some ETH for gas.

3. Send some ETH to the address of the **signer key** for gas. On each chain specified in `$HOME/.sgnd/config/cbridge.toml`, send corresponding native tokens to the signer key as gas is required for relaying
cBridge cross-chain requests.

4. Initialize the validator. Here we set a commission rate of 6% and a minimal self delegation of 10000 CELR tokens.

```sh
sgnd ops init-validator --commission-rate 0.06 --min-self-delegation 10000 --keystore ~/.sgnd/eth-ks/val.json --passphrase <val-ks-passphrase>
```

Note that it will take some time for the existing SGN validators to sync your new validator. Afterwards, verify your validator status:

```
sgnd query staking validator <val-eth-address>
```

You should see that your validator has a `tokens` field matching your delegated CELR amount. Make a note of the `consensus_address` - the address prefixed with `sgnvalcons`.

5. Update validator description:

```sh
echo $COSMOS_KEYRING_PASSPHRASE | sgnd tx staking edit-description --website "your-website" --contact "email-address"
```

Note that `COSMOS_KEYRING_PASSPHRASE` here is the passphrase for the keyring used in `sgnd keys add`, not the ones for the Ethereum JSON files.

After a while, verify the updated description:

```
sgnd query staking validator <val-eth-address>
```

6. To become a bonded validator, you need to delegate more CELR tokens to the validator:

```sh
sgnd ops delegate --validator <val-eth-address> --amount 50000 --keystore ~/.sgnd/eth-ks/val.json --passphrase <val-ks-passphrase>
```

After a while, verify the status:

```
sgnd query staking validator <val-eth-address>
```

If you have delegated enough tokens to qualify as a bonded validator, you should see that your validator has the status of `BOND_STATUS_BONDED`.

You can verify that your validator is in the Tendermint validator set:

```
sgnd query tendermint-validator-set
```

You should see an entry with `address` matching the `consensus_address` obtained in step 4.

You can also verify the delegation:

```
sgnd query staking delegation <val-eth-address> <val-eth-address>
```
