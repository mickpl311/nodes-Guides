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
SOOON
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

# Binary   04.10.23
```python 
cd $HOME && mkdir -p go/bin/
curl -s https://get.nibiru.fi/@v1.1.0! | bash
mv /usr/local/bin/nibid $HOME/go/bin
```
`nibid version --long | grep -e version -e commit`
+ 1.1.0
+ commit: 3d72b4f3eb17fa3a3f6006b19d8e5d868f392223

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
