<!-- START_TABLE -->
| 🛡Trusted Delegations🛡 | Token price🧲 | 💰Result in USD💰 |
|-------------|---------|---------------|
| 5805.9 | 0.365345 | 2121.159511965 |

<!-- END_TABLE -->



















[🔥OUR VALIDATOR🔥](https://restake.app/nibiru/nibivaloper1fn3tnv6nuhkp47qvyhxpvgy9jr42y8576t7wm4)
=

<h1 align="center"> Nibiru Mainnet</h1>

# Nibiru Mainnet  guide

![nibiru](https://user-images.githubusercontent.com/44331529/199216266-6b0da979-44a2-43e4-b9ef-de3a7c361b17.png)

[Website](https://nibiru.fi/) \
[GitHub](https://github.com/NibiruChain)
=
[EXPLORER 1](https://explorer.stavr.tech/Nibiru-Mainnet) \
[EXPLORER 2](https://exp.utsa.tech/nibiru)
=
- **Approximate hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8| 16GB | 300GB    |

# 1) Auto_install script
```python 
wget -O nibidm https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nibiru/nibidm && chmod +x nibidm && ./nibidm
```
# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu htop screen unzip bc fail2ban htop -y
```

## GO 1.21.6 (one command) 
```python
ver="1.21.6"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Binary   08.04.24
```python 
cd $HOME && mkdir -p go/bin/
git clone https://github.com/NibiruChain/nibiru/
cd $HOME/nibiru/
git fetch --all
git checkout v1.2.0
make install
```

*******🟢UPDATE🟢******* 08.04.24
```python
git clone https://github.com/NibiruChain/nibiru/
cd $HOME/nibiru/
git fetch --all
git checkout v1.2.0
make install
nibid version --long | grep -e commit -e version
#version: v1.2.0
#commit: 1ba22a79e36e7ed77e2e4503e72bfbbe5c609aa8
systemctl restart nibid && journalctl -fu nibid -o cat
```

`nibid version --long | grep -e version -e commit`
+ version: 1.2.0
+ commit:  1ba22a79e36e7ed77e2e4503e72bfbbe5c609aa8

## Initialisation
```python
nibid init STAVR_guide --chain-id=cataclysm-1
nibid config chain-id cataclysm-1

```
## Add wallet
```python
nibid keys add <walletName>
nibid keys add <walletName> --recover
```
# Genesis
```python
wget -O $HOME/.nibid/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nibiru/genesis.json"
```

`sha256sum $HOME/.nibid/config/genesis.json`
- 66c3bf943254a7d698849d201e0b7ae1ba7a94118b73f19916727742e26efd99  genesis.json

### Pruning
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.nibid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.nibid/config/app.toml
```
### Indexer (optional) one command
```python
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nibid/config/config.toml
```

### Set up the minimum gas price and Peers/Seeds/Filter peers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025unibi\"/;" ~/.nibid/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.nibid/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.nibid/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nibid/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.nibid/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.nibid/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.nibid/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.nibid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nibiru/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

# StateSync Nibiru Mainnet
```python
SNAP_RPC=https://nibiru.rpc.m.stavr.tech:443
peers="c416d67c3dbb2d30b803611469e6d2634099292d@nibiru.seed.stavr.tech:11036"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nibid/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nibid/config/config.toml
nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book
systemctl restart nibid && journalctl -fu nibid -o cat
```
# SnapShot Mainnet updated every 5 hours  
```python
cd $HOME
snap install lz4
sudo systemctl stop nibid
cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
rm -rf $HOME/.nibid/data
curl -o - -L https://nibiru.snapshot.stavr.tech/nibid-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.nibid --strip-components 2
mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json
wget -O $HOME/.nibid/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Nibiru/addrbook.json"
sudo systemctl restart nibid && journalctl -u nibid -f -o cat
```


# Start node (one command)
```python
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f -o cat
```

## Create validator
```
nibid tx staking create-validator \
--amount=1000000unibi \
--pubkey=$(nibid tendermint show-validator) \
--moniker=STAVR_guide \
--chain-id=cataclysm-1 \
--commission-rate="0.05" \
--commission-max-rate="0.2" \
--commission-max-change-rate="0.1" \
--min-self-delegation="1" \
--from=<walletname> \
--fees 5000unibi \
--identity="" \
--details="" \
--website="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Nibiru)
=

### Delete node (one command)
```python
sudo systemctl stop nibid
sudo systemctl disable nibid
rm /etc/systemd/system/nibid.service
sudo systemctl daemon-reload
cd $HOME
rm -rf .nibid
rm -rf nibiru
rm -rf $(which nibid)
```
#
### Sync Info
```python
nibid status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
nibid status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -u nibid -f -o cat
```
### Check Balance
```python
nibid query bank balances nibi...addressnibi1yjgn7z09ua9vms259j
```

# price feeder (✔️Oracle)


```python
mkdir -p $HOME/.nibid/pricefeeder && cd $HOME/.nibid/pricefeeder
wget https://github.com/NibiruChain/pricefeeder/releases/download/v1.0.1/pricefeeder_1.0.1_linux_amd64.tar.gz
tar -xvf pricefeeder_1.0.1_linux_amd64.tar.gz
mv pricefeeder /usr/local/bin/
```
`sha256sum /usr/local/bin/pricefeeder`
- 3448d8781fbda787dad42a075c0115e87cca8976737693310d14eec5bac54dff

## Create and fund a wallet 
`(your validator must be synced and in the active set)`
```python
nibid keys add nibiru_pf_wallet
```
```python
export CHAIN_ID="cataclysm-1"
export GRPC_ENDPOINT="localhost:9090"
export WEBSOCKET_ENDPOINT="ws://localhost:26657/websocket"
export EXCHANGE_SYMBOLS_MAP='{"bitfinex":{"ubtc:unusd":"tBTCUSD","ubtc:uusd":"tBTCUSD","ueth:unusd":"tETHUSD","ueth:uusd":"tETHUSD","uusdc:uusd":"tUDCUSD","uusdc:unusd":"tUDCUSD"},"coingecko":{"ubtc:uusd":"bitcoin","ubtc:unusd":"bitcoin","ueth:uusd":"ethereum","ueth:unusd":"ethereum","uusdt:uusd":"tether","uusdt:unusd":"tether","uusdc:uusd":"usd-coin","uusdc:unusd":"usd-coin","uatom:uusd":"cosmos","uatom:unusd":"cosmos","ubnb:uusd":"binancecoin","ubnb:unusd":"binancecoin","uavax:uusd":"avalanche-2","uavax:unusd":"avalanche-2","usol:uusd":"solana","usol:unusd":"solana","uada:uusd":"cardano","uada:unusd":"cardano"}}'
export FEEDER_MNEMONIC="<your mnemonic here>"
export VALIDATOR_ADDRESS="nibi1valoper..."
```
## Create a service file
```python
tee /etc/systemd/system/pricefeeder.service<<EOF
[Unit]
Description=Nibiru Pricefeeder
Requires=network-online.target
After=network-online.target

[Service]
Type=exec
User=$USER
ExecStart=/usr/local/bin/pricefeeder
Restart=on-failure
ExecReload=/bin/kill -HUP $MAINPID
KillSignal=SIGTERM
PermissionsStartOnly=true
LimitNOFILE=65535
Environment=CHAIN_ID='$CHAIN_ID'
Environment=GRPC_ENDPOINT='$GRPC_ENDPOINT'
Environment=WEBSOCKET_ENDPOINT='$WEBSOCKET_ENDPOINT'
Environment=EXCHANGE_SYMBOLS_MAP='$EXCHANGE_SYMBOLS_MAP'
Environment=FEEDER_MNEMONIC='$FEEDER_MNEMONIC'
Environment=VALIDATOR_ADDRESS='$VALIDATOR_ADDRESS'

[Install]
WantedBy=multi-user.target
EOF
```
## Start
```python
systemctl daemon-reload
systemctl enable pricefeeder
systemctl restart pricefeeder && journalctl -u pricefeeder -f -o cat
```

## Linking the wallet
```python
nibid tx oracle set-feeder <FEEDER_WALLET_ADDRESS> --from <name_vallet> --fees 5000unibi -y
```


### View a list of validators, which are Oracle
```python
nibid q oracle aggregate-votes | grep 'voter'
```
### VotePeriods
```python
nibid q account <FEEDER_WALLET_ADDRESS> -o json | jq -r .sequence
    OR
nibid q txs --events='message.sender=<FEEDER_WALLET_ADDRESS>&message.action=/nibiru.oracle.v1beta1.MsgAggregateExchangeRateVote' -oj|jq -r '.total_count'
```
### View Pricefeeder passes
```python
nibid q oracle miss <nibivaloper1...>
```
## Delete Pricefeeder
```python
systemctl stop pricefeeder
systemctl disable pricefeeder
rm /etc/systemd/system/pricefeeder.service
systemctl daemon-reload
cd $HOME
rm /usr/local/bin/pricefeeder
```
