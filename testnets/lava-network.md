# Lava Network

Lava Network is a centralized RPC node dependency solution built on top of Cosmos and Tendermint. Lava Network creates a decentralized infrastructure layer for blockchain remote procedure calls, RPC - remote procedure calls. In essence, this is a peer-to-peer RPC network.

This solution will allow Web3 companies to eliminate dependence on centralized RPC nodes, which will reduce vulnerability and the likelihood of being compromised.

### Update <a href="#t7co" id="t7co"></a>

```
cd $HOME/lava
git fetch --all 
git checkout v0.6.0
make install
sudo mv $HOME/go/bin/lavad /usr/local/bin/lavad
```

Check version 0.6.0

```
lavad version
```

Restart node and check logs

```
sudo systemctl restart lavad && sudo journalctl -fu lavad -o cat
```

### Node <a href="#aoos" id="aoos"></a>

```
# Update the repositories
apt update && apt upgrade -y
```

```
# Install developer packages
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

```
# Install Go (one command)
ver="1.19.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version

# go version go1.19.1 linux/amd64
```

```
# Set the variables

# Come up with the name of your node and replace it instead <your_moniker>
MONIKER_LAVA=<your_moniker>

echo 'export MONIKER_LAVA='$MONIKER_LAVA >> $HOME/.bash_profile
echo "export CHAIN_ID_LAVA=lava-testnet-1" >> $HOME/.bash_profile
echo "export PORT_LAVA=33" >> $HOME/.bash_profile
source $HOME/.bash_profile
# check whether the last command was executed
```

```
# Download binary files
cd $HOME 
git clone https://github.com/lavanet/lava 
cd lava 
git fetch --all 
git checkout v0.4.4
make install
sudo mv $HOME/go/bin/lavad /usr/local/bin/lavad
lavad version --long | grep -e version -e commit
# 0.4.4
```

```
# Initialize the node
lavad init $MONIKER_LAVA --chain-id $CHAIN_ID_LAVA
```

```
# Download Genesis
curl -s https://raw.githubusercontent.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T/main/testnet-1/genesis_json/genesis.json > $HOME/.lava/config/genesis.json

# Check Genesis
sha256sum ~/.lava/config/genesis.json
# 72170a8a7314cb79bc57a60c1b920e26457769667ce5c2ff0595b342c0080d78
```

```
# Set the ports

# config.toml
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT_LAVA}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT_LAVA}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT_LAVA}061\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT_LAVA}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT_LAVA}660\"%" $HOME/.lava/config/config.toml

# app.toml
sed -i.bak -e "s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT_LAVA}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT_LAVA}91\"%; s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:1${PORT_LAVA}7\"%" $HOME/.lava/config/app.toml

# client.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:${PORT_LAVA}657\"%" $HOME/.lava/config/client.toml

external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:${PORT_LAVA}656\"/" $HOME/.lava/config/config.toml
```

#### Setup config <a href="#grur" id="grur"></a>

```
# correct config (so we can no longer use the chain-id flag for every CLI command in client.toml)
lavad config chain-id lava-testnet-1

# adjust if necessary keyring-backend в client.toml 
lavad config keyring-backend os

lavad config node tcp://localhost:${PORT_LAVA}657

# Set the minimum price for gas
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ulava\"/;" ~/.lava/config/app.toml

# Add seeds/peers в config.toml
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.lava/config/config.toml

peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lava/config/config.toml

seeds="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.lava/config/config.toml

# Set up filter for "bad" peers
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.lava/config/config.toml

# https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T/blob/main/testnet-1/default_lavad_config_files/config.toml
sed -i -e "s/^timeout_commit *=.*/timeout_commit = \"60s\"/" $HOME/.lava/config/config.toml
sed -i -e "s/^timeout_propose *=.*/timeout_propose = \"60s\"/" $HOME/.lava/config/config.toml
sed -i -e "s/^create_empty_blocks_interval *=.*/create_empty_blocks_interval = \"60s\"/" $HOME/.lava/config/config.toml
sed -i 's/create_empty_blocks = .*/create_empty_blocks = true/g' ~/.lava/config/config.toml
sed -i 's/timeout_broadcast_tx_commit = ".*s"/timeout_broadcast_tx_commit = "601s"/g' ~/.lava/config/config.toml

# Set up pruning
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lava/config/app.toml
```

(OPTIONAL) Turn off indexing in config.toml

```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.lava/config/config.toml
```

SnapShot

```
snapshot_interval=1000
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.lava/config/app.toml

# install lz4
apt update
apt install snapd -y
snap install lz4

SNAP_LAVA=$(curl -s https://snapshots1-testnet.nodejumper.io/lava-testnet/ | egrep -o ">lava-testnet-1.*\.tar.lz4" | tr -d ">")
curl https://snapshots1-testnet.nodejumper.io/lava-testnet/${SNAP_LAVA} | lz4 -dc - | tar -xf - -C $HOME/.lava
```

**StateSync**

```
peers="433be6210ad6350bebebad68ec50d3e0d90cb305@217.13.223.167:60856"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lava/config/config.toml

SNAP_RPC=https://t-lava.rpc.utsa.tech:443

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.lava/config/config.toml

systemctl restart lavad && journalctl -u lavad -f -o cat
```

**Cosmovisor**

```
# Install Cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

```
# Create directories
mkdir -p ~/.lava/cosmovisor
mkdir -p ~/.lava/cosmovisor/genesis
mkdir -p ~/.lava/cosmovisor/genesis/bin
mkdir -p ~/.lava/cosmovisor/upgrades
```

```
# Copy the binary file to the cosmovisor folder
cp `which lavad` ~/.lava/cosmovisor/genesis/bin/lavad
```

```
# Create service file (One command)
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=lava daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants
Restart=always
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_NAME=lavad"
Environment="DAEMON_HOME=${HOME}/.lavad"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"

[Install]
WantedBy=multi-user.target
EOF
```

```
# Start the node
systemctl daemon-reload
systemctl enable lavad
systemctl restart lavad && journalctl -u lavad -f -o cat

# Escape from logs ctrl+c
```

```
# Check the logs again
journalctl -u lavad -f -o cat

# Escape from logs ctrl+c
```

If you cannot connect to peers long time download addrbook.

```
wget -O $HOME/.lava/config/addrbook.json "https://share2.utsa.tech/lava/addrbook.json"

systemctl restart lavad && journalctl -u lavad -f -o cat
```

Now all ok, check the status

```
curl localhost:${PORT_LAVA}657/status
```

"catching\_up": false means that the node is synchronized, we are waiting for complete synchronization

### Wallet <a href="#2osy" id="2osy"></a>

```
# Create wallet
lavad keys add wallet
```

Create a password for the wallet and write it down so you don't forget it. The wallet has been created. In the last line there will be a phrase that must be written down

```
# If the wallet was already there, restore it
lavad keys add wallet --recover
# Insert the seed phrase from your wallet
# If everything is correct, you will see your wallet data
```

Go to the [# ](https://discord.com/channels/947911971515293759/984840062871175219)[faucet](https://discord.com/channels/963778337904427018/1059851367717556314) branch and request tokens

```
# Save the wallet address
# Replace <your_address> with your wallet address
WALLET_LAVA=<your_address>
echo "export WALLET_LAVA="${WALLET_LAVA}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
# Check balance
lavad q bank balances $WALLET_LAVA
```

### Validator <a href="#zris" id="zris"></a>

Do not forget to create a profile on https://keybase.io/ and set a profile photo there that will be imported by key and used for your validators.

```
lavad tx staking create-validator \
--amount 9000000ulava \
--from=$WALLET_LAVA \
--commission-rate "0.07" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(lavad tendermint show-validator) \
--moniker=$MONIKER_LAVA \
--chain-id=$CHAIN_ID_LAVA \
--fees=10000ulava \
--identity="" \
--details="" \
--website="" \
-y
```

Check yourself in the list [explorer](https://lava.explorers.guru/validators)

Or by command

```
lavad query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq
```

```
# Edit the validator
lavad tx staking edit-validator \
  --new-moniker=$MONIKER_LAVA \
  --website="" \
  --identity=<identity> \
  --details="" \
  --chain-id=$CHAIN_ID_LAVA \
  --fees=10000ulava \
  --from=wallet
  
```

```
# Save valoper_address in bash
# Change <your_valoper_address> to the address of the validator, starting with lavavaloper...
VALOPER_LAVA=<your_valoper_address>
echo "export VALOPER_LAVA="${VALOPER_LAVA}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

!!! Save priv\_validator\_key.json which is located in /root/.gitopiad/config

### Information <a href="#m7xp" id="m7xp"></a>

```
# Check bloks
lavad status 2>&1 | jq ."SyncInfo"."latest_block_height"

# Check the logs
journalctl -u lavad -f -o cat

# Check status
curl localhost:${PORT_LAVA}657/status

# Check balance
lavad q bank balances $WALLET_LAVA

# Check pubkey of validator
lavad tendermint show-validator

# Check validator
lavad query staking validator $VALOPER_LAVA
lavad query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq

# Check info of TX_HASH
lavad query tx <TX_HASH>

# Check how many blocks were passed by the validator and from which block the asset
lavad q slashing signing-info $(lavad tendermint show-validator)
```

```
Transactions

# Collect rewards from all validators delegated to them (without commission)
lavad tx distribution withdraw-all-rewards --from <name_wallet> --fees 5000ulava -y

# Collect rewards from a separate validator or rewards + commission from your own validator
lavad tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5000ulava --commission -y

# Delegate yourself (this is how 1 coin is sent)
lavad tx staking delegate <valoper_address> 1000000ulava --from <name_wallet> --fees 5000ulava -y

# Redelegate to other validator
lavad tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000ulava --from <name_wallet> --fees 5000ulava -y

# unbond 
lavad tx staking unbond <addr_valoper> 1000000ulava --from <name_wallet> --fees 5000ulava -y

# Send tokens to other adress
lavad tx bank send <name_wallet> <address> 1000000ulava --fees 5000ulava -y

# Escape from jail
lavad tx slashing unjail --from <name_wallet> --fees 5000ulava -y
```

! If the transactions are not sent with the error account sequence mismatch, expected 18, got 17: incorrect account sequence, then add the flag -s 18 to the command (replace the number with the one that is waiting for the sequence)

**Work with wallet**

```
# Check list of the wallets
lavad keys list

# Delete wallet
lavad keys delete <name_wallet>
```

**Delete the node**

```
systemctl stop lavad && \
systemctl disable lavad && \
rm /etc/systemd/system/lavad.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .lava GHFkqmTzpdNLDd6T lava && \
rm -rf $(which lavad)
```

Governance

```
# List of proposals
lavad q gov proposals

# Check vote result
lavad q gov proposals --voter <ADDRESS>

# Vote 
lavad tx gov vote 1 yes --from <name_wallet> --fees 555ulava

# Send deposit for proposal
lavad tx gov deposit 1 1000000ulava --from <name_wallet> --fees 555ulava
```

**Peers and RPC**

```
FOLDER=.lava

# Check your peer
PORTR=$(grep -A 3 "\[p2p\]" ~/$FOLDER/config/config.toml | egrep -o ":[0-9]+") && \
echo $(lavad tendermint show-node-id)@$(curl ifconfig.me)$PORTR

# Check your port RPC
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/$FOLDER/config/config.toml | egrep -o ":[0-9]+")\033[0m"

# Check quantity of ports
PORT=
curl -s http://localhost:$PORT/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr | split(":")[2])"' | wc -l
```

### We are a small group of enthusiasts who participate in testnet and validate projects on the mainet, invest in promising projects. <a href="#iiry" id="iiry"></a>

#### Discord - [https://discord.gg/cvhkpvsz](https://discord.gg/cvhkpvsz) <a href="#tptv" id="tptv"></a>

#### Telegram - <a href="#rjba" id="rjba"></a>
