[🔥OUR VALIDATOR🔥](https://restake.app/persistence/persistencevaloper1n4mw69thxqggt4p2j8t420nudn0gceve5kchck)
=

# Persistence Mainnet guide

![perst](https://github.com/obajay/nodes-Guides/assets/44331529/6fb9af89-d670-47f8-a196-e6f2aad8a1ea)

[WebSite](https://persistence.one/)\
[GitHub](https://github.com/persistenceOne/persistenceCore)
=
[EXPLORER_1](https://explorer.stavr.tech/Persistence-Mainnet) \
[EXPLORER_2](https://ping.pub/persistence/staking)
=

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |   8|  16GB | 250GB    |


# 1) Auto_install script
```python
wget -O persism https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Persistence/persism && chmod +x persism && ./persism
```

# 2) Manual installation

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

## GO 1.21.6
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

# Build 15.04.24
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/persistenceOne/persistenceCore
cd persistenceCore
git checkout v11.9.0-fh
make install
```
*******🟢UPDATE🟢******* 15.04.24
```python
cd $HOME/persistenceCore
git pull
git checkout v11.9.0-fh
make install
persistenceCore version --long | grep -e commit -e version
#commit: 1f9f3aa50cf16d9c04a58d59d06f001fe764b857
#version: v11.9.0-fh
sudo systemctl restart persistenceCore && sudo journalctl -fu persistenceCore -o cat
```

`persistenceCore version --long | head`
- version: v11.9.0-fh
- commit: 1f9f3aa50cf16d9c04a58d59d06f001fe764b857

```python
persistenceCore init STAVR_guide --chain-id core-1
persistenceCore config chain-id core-1
```    

## Create/recover wallet
```python
persistenceCore keys add <walletname>
  OR
persistenceCore keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.persistenceCore/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Persistence/genesis.json"
```
`sha256sum $HOME/.persistenceCore/config/genesis.json`
+ 673d30abd133a13210bf271d8a52aabc3f1b12c0864f543f4313f7f9589bdb53

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uxprt\"/;" ~/.persistenceCore/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.persistenceCore/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.persistenceCore/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.persistenceCore/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.persistenceCore/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.persistenceCore/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.persistenceCore/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.persistenceCore/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.persistenceCore/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.persistenceCore/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.persistenceCore/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.persistenceCore/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Persistence/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/persistenceCore.service > /dev/null <<EOF
[Unit]
Description=persistenceCore
After=network-online.target

[Service]
User=$USER
ExecStart=$(which persistenceCore) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Persistence Mainnet
```python
SNAP_RPC=https://persistence.rpc.m.stavr.tech:443
peers="ab7fc0b9b3c523dacec0500c9f9f1f7f4699d551@persis-m.seed.stavr.tech:4056"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.persistenceCore/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.persistenceCore/config/config.toml
persistenceCore tendermint unsafe-reset-all --home /root/.persistenceCore
curl -o - -L https://persis-m.wasm.stavr.tech/wasm-persis.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.persistenceCore --strip-components 2
wget -O $HOME/.persistenceCore/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Persistence/addrbook.json"
systemctl restart persistenceCore && journalctl -fu persistenceCore -o cat
```
# SnapShot Mainnet updated every 5 hours  
```python
cd $HOME
apt install lz4
sudo systemctl stop persistenceCore
cp $HOME/.persistenceCore/data/priv_validator_state.json $HOME/.persistenceCore/priv_validator_state.json.backup
rm -rf $HOME/.persistenceCore/data
curl -o - -L https://persis-m.snapshot.stavr.tech/persis-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.persistenceCore --strip-components 2
curl -o - -L https://persis-m.wasm.stavr.tech/wasm-persis.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.persistenceCore --strip-components 2
mv $HOME/.persistenceCore/priv_validator_state.json.backup $HOME/.persistenceCore/data/priv_validator_state.json
wget -O $HOME/.persistenceCore/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Persistence/addrbook.json"
sudo systemctl restart persistenceCore && journalctl -fu persistenceCore -o cat
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable persistenceCore
sudo systemctl restart persistenceCore && sudo journalctl -fu persistenceCore -o cat
```

### Create validator
```python
persistenceCore tx staking create-validator \
--commission-rate 0.1 \
--commission-max-rate 0.1 \
--commission-max-change-rate 0.1 \
--min-self-delegation "1" \
--amount 1000000uxprt \
--pubkey $(persistenceCore tendermint show-validator) \
--from <wallet> \
--moniker="STAVR_guide" \
--chain-id core-1 \
--gas 300000 \
--identity="" \
--website="" \
--details="" -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Persistence)
=


## Delete node
```python
sudo systemctl stop persistenceCore
sudo systemctl disable persistenceCore
rm /etc/systemd/system/persistenceCore.service
sudo systemctl daemon-reload
cd $HOME
rm -rf persistenceCore
rm -rf .persistenceCore
rm -rf $(which persistenceCore)
```
#
### Sync Info
```python
persistenceCore status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
persistenceCore status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -fu persistenceCore -o cat
```
### Check Balance
```python
persistenceCore query bank balances persistence...addressjkl1yjgn7z09ua9vms259j
```


<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
persistenceCore status 2>&1 | jq .NodeInfo
persistenceCore status 2>&1 | jq .SyncInfo
persistenceCore status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu persistenceCore -o cat
```
#### Check service status
```python
sudo systemctl status persistenceCore
```
#### Restart service
```python
sudo systemctl restart persistenceCore
```
#### Stop service
```python
sudo systemctl stop persistenceCore
```
#### Start service
```python
sudo systemctl start persistenceCore
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable persistenceCore
sudo systemctl enable persistenceCore
```
#### Your Peer
```python
echo $(persistenceCore tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.persistenceCore/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
persistenceCore keys add Wallet_Name
      OR
persistenceCore keys add Wallet_Name --recover
```
#### Check all keys
```python
persistenceCore keys list
```
#### Check Balance
```python
persistenceCore query bank balances persistence...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
persistenceCore keys delete Wallet_Name
```
#### Export Key
```python
persistenceCore keys export wallet
```
#### Import Key
```python
persistenceCore keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
persistenceCore tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id core-1 \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 -y
```

#### Your Valoper-Address
```python
persistenceCore keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
persistenceCore tendermint show-address
```
#### Your Validator-Info
```python
persistenceCore query staking validator persistencevaloperaddress......
```
#### Jail Info
```python
persistenceCore query slashing signing-info $(persistenceCore tendermint show-validator)
```
#### Unjail
```python
persistenceCore tx slashing unjail --from Wallet_name --chain-id core-1 --gas 350000 -y
```
#### Active Validators List
```python
persistenceCore q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
persistenceCore q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(persistenceCore  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(persistenceCore status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
persistenceCore tx distribution withdraw-all-rewards --from Wallet_Name --chain-id core-1 --gas 350000 -y
```
#### Withdraw and commission from your Validator
```python
persistenceCore tx distribution withdraw-rewards persistencevaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e --from Wallet_Name --gas 350000 --chain-id=core-1 --commission -y
```
#### Delegate tokens to your validator
```python
persistenceCore tx staking delegate Your_persistencevalpoer........ "100000000"uxprt --from Wallet_Name --gas 350000 --chain-id=core-1 -y
```
#### Delegate tokens to different validator
```python
persistenceCore tx staking delegate persistencevalpoer........ "100000000"uxprt --from Wallet_Name --gas 350000 --chain-id=core-1 -y
```
#### Redelegate tokens to another validator
```python
persistenceCore tx staking redelegate Your_persistencevalpoer........ persistencevalpoer........ "100000000"uxprt --from Wallet_Name --gas 350000  --chain-id=core-1 -y
```

#### Unbond tokens from your validator or different validator
```python
persistenceCore tx staking unbond Your_persistencevalpoer........ "100000000"uxprt --from Wallet_Name --gas 350000 --chain-id=core-1 -y
persistenceCore tx staking unbond persistencevalpoer........ "100000000"uxprt --from Wallet_Name --gas 350000 --chain-id=core-1 -y
```

#### Transfer tokens from wallet to wallet
```python
persistenceCore tx bank send Your_persistenceaddress............ persistenceaddress........... "1000000000000000000"uxprt --gas 350000 --chain-id=core-1 -y
```

# 📝Governance

#### View all proposals
```python
persistenceCore query gov proposals
```

#### View specific proposal
```python
persistenceCore query gov proposal 1
```

#### Vote yes
```python
persistenceCore tx gov vote 1 yes --from Wallet_Name --gas 350000  --chain-id=core-1 -y
```
#### Vote no
```python
persistenceCore tx gov vote 1 no --from Wallet_Name --gas 350000  --chain-id=core-1 -y
```
#### Vote abstain
```python
persistenceCore tx gov vote 1 abstain --from Wallet_Name --gas 350000  --chain-id=core-1 -y
```
#### Vote no_with_veto
```python
persistenceCore tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000  --chain-id=core-1 -y
```
