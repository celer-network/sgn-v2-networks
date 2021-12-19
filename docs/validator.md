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

1. Start an EC2 machine with the Ubuntu 20.04 LTS image. We recommend using `t3.medium` (2 vCPUs, 4GB RAM and 5Gbps network bandwidth) with an EBS volume of at least
100GB for testnet and `c5a.2xlarge` (8 vCPUs, 16GB RAM and 10Gbps network bandwidth) with an EBS volume of at least 500GB for mainnet. Use the appropriate security
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

```sh
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

## Setup binary, config and accounts

1. From the `/home/ubuntu` directory, download and install the `sgnd` binary:

```sh
curl https://github.com/celer-network/sgn-v2-networks/releases/download/v1.4.1/sgn-v1.4.1-linux-amd64.tar.gz | tar -xz
mv sgnd $GOBIN
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

Backup the generated Tendermint key files `$HOME/.sgnd/config/node_key.json` and `$HOME/.sgnd/config/priv_validator_key.json` securely. Make sure the keys are **never** committed to any repo.

5. Fill out the `moniker` field in `config/config.toml` with your `node-name`.
Fill out the `external_address` field with `<public-ip:26656>`, where the `public-ip` is the public IP of the EC2 machine hosting the node. Currently, the Celer foundation nodes restrict the access to port 26656, so please **report your public IP to the Celer team** to get whitelisted.

6. Add a Cosmos SDK / Tendermint validator account:

```sh
sgnd keys add <node-name>
```

Input a passphrase for the keyring. Backup the passphrase along with the displayed mnemonic phrase securely. Make sure they are **never** committed to any repo.

To view the account created, run:

```sh
sgnd keys list
```

Make a note of the **sgn-prefixed account address**.

7. Prepare an Ethereum gateway URL. You can use services like [Infura](https://infura.io/),
[Alchemy](https://alchemyapi.io/) or run your own node.

8. Prepare an Ethereum key as the **validator key**, which will be used for initializing the validator and occasional operations such as withdrawing rewards. Therefore, the validator key does not need to stay online.

The validator key can be prepared via two ways:

### Using a local keystore JSON file

Currently, the advantage of a local keystore JSON is that operations can be done via the command line. However, it will require saving a passphrase in clear text on the machine running the node, so please be careful about access control.

Assuming `geth` is installed, the keystore JSON file can be generated via:

```sh
geth account new
```

Backup the passphrase securely. Save the JSON file as `$HOME/.sgnd/eth-ks/val.json`:

```sh
mkdir $HOME/.sgnd/eth-ks
cp <path-to-keystore-json> $HOME/.sgnd/eth-ks/val.json
```

### Using MetaMask / hardware wallet

This approach is more secure than a local keystore, but it will require interacting with the staking contract via Etherscan.

Please reserve a dedicated account on MetaMask. If using a hardware wallet, make sure it is compatible with MetaMask.

9. Prepare another Ethereum key as the **signer key**, which will be used for signing
cross-chain transactions and needs to stay online. Currently, the signer key can only be provided as a local keystore JSON file. Save it as `$HOME/.sgnd/eth-ks/signer.json`.

10. Fill in the fields in `$HOME/.sgnd/config/sgn.toml` with the correct values:

    | Field | Description |
    | ----- | ----------- |
    | eth.gateway | The Ethereum gateway URL obtained from step 7 |
    | eth.signer_keystore | The path to the signer Ethereum keystore file in step 9 |
    | eth.signer_passphrase | The passphrase of the signer keystore |
    | eth.validator_address | The **Ethereum address** of the validator key prepared in step 8 |
    | sgnd.passphrase | The **Cosmos keyring passphrase** you typed in step 6 |
    | sgnd.validator_account | The **sgn-prefixed validator Cosmos SDK account** added in step 6 |

11. Fill in the missing gateway URLs in `$HOME/.sgnd/config/cbridge.toml` with the corresponding JSON-RPC URLs for the chains. In general, we recommend using paid provider services instead of the public endpoints for better reliability.

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
[faucet](https://faucet.paradigm.xyz/). Contact the Celer team for some Goerli test CELR tokens.

For mainnet, prepare real ETH and CELR tokens.

For both networks, contact the Celer team to get whitelisted for a validator spot.

2. Send ETH and CELR to the address of the **validator key**. Make sure it has enough CELR for the intended self delegation and some ETH for gas.

3. Send some ETH to the address of the **signer key** for gas. On each chain specified in `$HOME/.sgnd/config/cbridge.toml`, send corresponding native tokens to the signer key as gas is required for relaying
cBridge cross-chain requests.

4. Initialize the validator. Here we set a commission rate of 6% and a minimal self delegation of 10000 CELR tokens.

### For validator key on local keystore JSON file

Initialize using the command line:

```sh
sgnd ops init-validator --commission-rate 0.06 --min-self-delegation 10000 --keystore ~/.sgnd/eth-ks/val.json --passphrase <val-ks-passphrase>
```

### For validator key on MetaMask / hardware wallet

First, approve CELR tokens to the `Staking` contract:

i. Go to [Etherscan](etherscan.io) for either Goerli or mainnet, search for the **celr** contract address taken from `eth.contract_addresses` in `$HOME/.sgnd/sgn.toml`. You should see the verified `CelerToken` contract on the corresponding network.

ii. Make sure the validator key address is selected on your MetaMask / hardware wallet. Click on the "Write Contract" button under the "Contract" tab, click on "Connect to Web3" to connect your wallet.

iii. Find the `approve` method.

For `spender`, put the address of the **staking** contract address taken from `eth.contract_addresses` in `$HOME/.sgnd/sgn.toml`.

For `amount`, put at least the amount of CELR tokens intended for self delegation times the decimals of CELR, which is 1e18. Eg. `10000000000000000000000` for 10000 CELR. Feel free to approve more.

Click "Write" and send the transaction.

Then, initialize the validator with a self delegation:

i. Find the `Staking` contract on Etherscan and connect your wallet to "Write Contract".

ii. Find the `initializeValidator` method.

For `signer`, put the address of the **signer key**.

For `minSelfDelegation`, put the amount of CELR tokens for self delegation times 1e18.

For `commissionRate`, put the intended commission rate times 10000. Eg. 600 would represent 6%.

Click "Write" and send the transaction.

Note that it will take some time for the existing SGN validators to sync your new validator. Afterwards, verify your validator status:

```sh
sgnd query staking validator <val-eth-address>
```

You should see that your validator has a `tokens` field matching your delegated CELR amount. Make a note of the `consensus_address` - the address prefixed with `sgnvalcons`.

5. Update validator description:

```sh
echo $COSMOS_KEYRING_PASSPHRASE | sgnd tx staking edit-description --website "your-website" --contact "email-address"
```

Note that `COSMOS_KEYRING_PASSPHRASE` here is the passphrase for the keyring used in `sgnd keys add`, not the ones for the Ethereum JSON files.

After a while, verify the updated description:

```sh
sgnd query staking validator <val-eth-address>
```

6. To become a bonded validator, your validator needs to have more CELR tokens delegated to it. Note that additional delegation does not need to come from the validator account, so feel free to use any key that holds CELR tokens.

### For local keystore JSON file

```sh
sgnd ops delegate --validator <val-eth-address> --amount 50000 --keystore <path-to-keystore-file> --passphrase <ks-passphrase>
```

### For MetaMask / hardware wallet

First, approve CELR tokens to the `Staking` contract following the instructions in step 4.

Then, delegate to the validator:

i. Find the staking contract on Etherscan and connect your wallet to "Write Contract".

ii. Find the `delegate` method.

For `valAddr`, put the address of the **validator key**.

For `tokens`, put the amount of CELR tokens to delegate times 1e18.

Click "Write" and send the transaction.

After a while, verify the status:

```sh
sgnd query staking validator <val-eth-address>
```

If you have delegated enough tokens to qualify as a bonded validator, you should see that your validator has the status of `BOND_STATUS_BONDED`.

You can verify that your validator is in the Tendermint validator set:

```sh
sgnd query tendermint-validator-set
```

You should see an entry with `address` matching the `consensus_address` obtained in step 4.

You can also verify the delegation:

```sh
sgnd query staking delegation <val-eth-address> <val-eth-address>
```
