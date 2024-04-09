# OG Testnet guide

![og](https://github.com/obajay/nodes-Guides/assets/44331529/87208620-9a36-4eea-9fdc-12437f49fbfc)

[WebSite](https://0g.ai/)\
[GitHub](https://github.com/0glabs/0g-evmos)
=
[EXPLORER](https://explorer.stavr.tech/Og-Testnet)
=

- **Аpproximate hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   8|  16GB | 250GB    |


# 1) Auto_install script
```python
wget -O ogt https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Og/ogt && chmod +x ogt && ./ogt
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

# Build 09.04.24
```python
cd $HOME && mkdir -p go/bin/
git clone https://github.com/0glabs/0g-evmos.git
cd 0g-evmos
git checkout v1.0.0-testnet
make build
mv $HOME/0g-evmos/build/ $HOME/go/bin/
```
*******🟢UPDATE🟢******* 00.00.23
```python
SOOON
```

`evmosd version --long | grep -e version -e commit`
- version: 1.0.0-testnet
- commit: b52c268d6a66949195bc89e2e2f6b755c4d0dd3b

```python
evmosd init STAVR_guide --chain-id zgtendermint_9000-1
evmosd config chain-id zgtendermint_9000-1
```    

## Create/recover wallet
```python
evmosd keys add <walletname>
  OR
evmosd keys add <walletname> --recover
```

## Download Genesis
```python
wget -O $HOME/.evmosd/config/genesis.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Og/genesis.json"

```
`sha256sum $HOME/.evmosd/config/genesis.json`
+ 0d471d89866e4bdb94647d198a24d0e1c64333a8f799707e35b959f2aed82026

## Set up the minimum gas price and Peers/Seeds/Filter peers/MaxPeers
```python
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0agnet\"/;" ~/.evmosd/config/app.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.evmosd/config/config.toml
peers=""
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.evmosd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.evmosd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.evmosd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.evmosd/config/config.toml

```
### Pruning (optional)
```python
pruning="custom"
pruning_keep_recent="1000"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.evmosd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.evmosd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.evmosd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.evmosd/config/app.toml
```
### Indexer (optional) 
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.evmosd/config/config.toml
```

## Download addrbook
```python
wget -O $HOME/.evmosd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Projects/Og/addrbook.json"
```

# Create a service file
```python
sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
[Unit]
Description=evmosd
After=network-online.target

[Service]
User=$USER
ExecStart=$(which evmosd) start --chain-id zgtendermint_9000-1
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
# StateSync Og Testnet
```python
SOOOOOOOOOON
```
# SnapShot Testnet updated every 5 hours  
```python
SOOOOOOOON
```

## Start
```python
sudo systemctl daemon-reload
sudo systemctl enable evmosd
sudo systemctl restart evmosd && sudo journalctl -u evmosd -f -o cat
```

### Create validator
```python
evmosd tx staking create-validator \
  --amount=10000000000000000aevmos \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker="STAVR_guide" \
  --chain-id=zgtendermint_9000-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.1 \
  --commission-max-change-rate=0.1 \
  --min-self-delegation=1 \
  --from=STAVR1 \
  --identity="" \
  --details="" \
  --website="" \
  --gas=500000 --gas-prices=99999aevmos \
  -y
```

[🧩Services and Tools🧩](https://github.com/obajay/StateSync-snapshots/tree/main/Projects/Og)
=


## Delete node
```python
sudo systemctl stop evmosd
sudo systemctl disable evmosd
rm /etc/systemd/system/evmosd.service
sudo systemctl daemon-reload
cd $HOME
rm -rf 0g-evmos
rm -rf .evmosd
rm -rf $(which evmosd)
```
#
### Sync Info
```python
evmosd status 2>&1 | jq .SyncInfo
```
### NodeINfo
```python
evmosd status 2>&1 | jq .NodeInfo
```
### Check node logs
```python
sudo journalctl -fu evmosd -o cat
```
### Check Balance
```python
evmosd query bank balances evmos...addressjkl1yjgn7z09ua9vms259j
```


<h1 align="center"> 📚Useful commands📚 </h1>

# ⚙️Service

#### Info
```python
evmosd status 2>&1 | jq .NodeInfo
evmosd status 2>&1 | jq .SyncInfo
evmosd status 2>&1 | jq .ValidatorInfo
```
#### Check node logs
```python
sudo journalctl -fu evmosd -o cat
```
#### Check service status
```python
sudo systemctl status evmosd
```
#### Restart service
```python
sudo systemctl restart evmosd
```
#### Stop service
```python
sudo systemctl stop evmosd
```
#### Start service
```python
sudo systemctl start evmosd
```
#### reload/disable/enable
```python
sudo systemctl daemon-reload
sudo systemctl disable evmosd
sudo systemctl enable evmosd
```
#### Your Peer
```python
echo $(evmosd tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.evmosd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

# 🥅Working with keys

#### New Key or Recover Key
```python
evmosd keys add Wallet_Name
      OR
evmosd keys add Wallet_Name --recover
```
#### Check all keys
```python
evmosd keys list
```
#### Check Balance
```python
evmosd query bank balances evmos...addressjkl1yjgn7z09ua9vms259j
```
#### Delete Key
```python
evmosd keys delete Wallet_Name
```
#### Export Key
```python
evmosd keys export wallet
```
#### Import Key
```python
evmosd keys import wallet wallet.backup
```

# 🚀Validator Management

#### Edit Validator
```python
evmosd tx staking edit-validator \
--new-moniker "Your_Moniker" \
--identity "Keybase_ID" \
--details "Your_Description" \
--website "Your_Website" \
--security-contact "Your_Email" \
--chain-id zgtendermint_9000-1 \
--commission-rate 0.05 \
--from Wallet_Name \
--gas 350000 -y
```

#### Your Valoper-Address
```python
evmosd keys show Wallet_Name --bech val
```
#### Your Valcons-Address
```python
evmosd tendermint show-address
```
#### Your Validator-Info
```python
evmosd query staking validator evmosvaloperaddress......
```
#### Jail Info
```python
evmosd query slashing signing-info $(evmosd tendermint show-validator)
```
#### Unjail
```python
evmosd tx slashing unjail --from Wallet_name --chain-id zgtendermint_9000-1 --gas 350000 -y
```
#### Active Validators List
```python
evmosd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Inactive Validators List
```python
evmosd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
#### Check that your key matches the validator  (Win - Good.  Lose - Bad)
```python
VALOPER=Enter_Your_valoper_Here
[[ $(evmosd  q staking validator $VALOPER -oj | jq -r .consensus_pubkey.key) = $(evmosd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\nYou win\n" || echo -e "\nYou lose\n"
```

#### Withdraw all rewards from all validators
```python
evmosd tx distribution withdraw-all-rewards --from Wallet_Name --chain-id zgtendermint_9000-1 --gas 350000 -y
```
#### Withdraw and commission from your Validator
```python
evmosd tx distribution withdraw-rewards evmosvaloper1amxp0k0hg4edrxg85v07t9ka2tfuhamhldgf8e --from Wallet_Name --gas 350000 --chain-id=zgtendermint_9000-1 --commission -y
```
#### Delegate tokens to your validator
```python
evmosd tx staking delegate Your_evmosvalpoer........ "100000000"aevmos --from Wallet_Name --gas 350000 --chain-id=zgtendermint_9000-1 -y
```
#### Delegate tokens to different validator
```python
evmosd tx staking delegate evmosvalpoer........ "100000000"aevmos --from Wallet_Name --gas 350000 --chain-id=zgtendermint_9000-1 -y
```
#### Redelegate tokens to another validator
```python
evmosd tx staking redelegate Your_evmosvalpoer........ evmosvalpoer........ "100000000"aevmos --from Wallet_Name --gas 350000  --chain-id=zgtendermint_9000-1 -y
```

#### Unbond tokens from your validator or different validator
```python
evmosd tx staking unbond Your_gevmosvalpoer........ "100000000"aevmos --from Wallet_Name --gas 350000 --chain-id=zgtendermint_9000-1 -y
evmosd tx staking unbond evmosvalpoer........ "100000000"aevmos --from Wallet_Name --gas 350000 --chain-id=zgtendermint_9000-1 -y
```

#### Transfer tokens from wallet to wallet
```python
evmosd tx bank send Your_evmosaddress............ evmosaddress........... "1000000000000000000"aevmos --gas 350000 --chain-id=zgtendermint_9000-1 -y
```

# 📝Governance

#### View all proposals
```python
evmosd query gov proposals
```

#### View specific proposal
```python
evmosd query gov proposal 1
```

#### Vote yes
```python
evmosd tx gov vote 1 yes --from Wallet_Name --gas 350000  --chain-id=zgtendermint_9000-1 -y
```
#### Vote no
```python
evmosd tx gov vote 1 no --from Wallet_Name --gas 350000  --chain-id=zgtendermint_9000-1 -y
```
#### Vote abstain
```python
evmosd tx gov vote 1 abstain --from Wallet_Name --gas 350000  --chain-id=zgtendermint_9000-1 -y
```
#### Vote no_with_veto
```python
evmosd tx gov vote 1 no_with_veto --from Wallet_Name --gas 350000  --chain-id=zgtendermint_9000-1 -y
```
