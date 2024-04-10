# Tangle Mainnet guide

![tangl](https://github.com/obajay/nodes-Guides/assets/44331529/78b39cb8-33de-4b55-ba66-852527e5a29d)


[WebSite](https://www.tangle.tools/)\
[GitHub](https://github.com/webb-tools/tangle/blob/d12e316d7c7f64fdcc76842e03944a43221285d8/chainspecs/testnet/tangle-standalone.json)
=
[EXPLORER](https://testnet-explorer.tangle.tools) \
[TELEMETRY](https://telemetry.polkadot.io/#list/0x44f68476df71ebf765b630bf08dc1e0fedb2bf614a1aa0563b3f74f20e47b3e0) \
[CHAIN](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.tangle.tools#/accounts)
=

- **Approximate hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Mainnet   |  16|  32GB| 250GB    |

### Preparing the server
```python
sudo apt update && sudo apt upgrade -y
apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

# Build 10.04.24
```python
cd $HOME
mkdir -p $HOME/.tangle && cd $HOME/.tangle
wget -O tangle https://github.com/webb-tools/tangle/releases/download/v1.0.0/tangle-default-linux-amd64 && chmod +x tangle
mv tangle /usr/bin/
wget -O $HOME/.tangle/tangle-mainnet.json "https://raw.githubusercontent.com/webb-tools/tangle/main/chainspecs/mainnet/tangle-mainnet.json"
chmod 744 ~/.tangle/tangle-mainnet.json
sha256sum ~/.tangle/tangle-mainnet.json
#b640e7fb959066ce29a3ddece42f17cc4e76b4383fd57e4f4249e2c80bff8a00
```

`tangle --version`
- tangle 1.0.0-2029231-x86_64-linux-gnu

# Create a service file
```python
yourname=<name>
```
```python
sudo tee /etc/systemd/system/tangle.service > /dev/null << EOF
[Unit]
Description=Tangle Validator Node
After=network-online.target
StartLimitIntervalSec=0
[Service]
User=$USER
Restart=always
RestartSec=3
ExecStart=/usr/bin/tangle \
  --base-path $HOME/.tangle \
  --name '$yourname' \
  --chain tangle-mainnet \
    --node-key-file "/home/$yourname/node-key" \
  --port 30333 \
  --rpc-port 9933 \
  --prometheus-port 9515 \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
  --validator \
  --pruning archive \
  --no-mdns
[Install]
WantedBy=multi-user.target
EOF
```

## Start
```python
systemctl daemon-reload
systemctl enable tangle
systemctl restart tangle && journalctl -u tangle -f -o cat
```

- After launch, we wait for our node to synchronize. You can track our condition using telemetry

[TELEMETRY](https://telemetry.polkadot.io/#list/0x44f68476df71ebf765b630bf08dc1e0fedb2bf614a1aa0563b3f74f20e47b3e0)
=

- After the node has synchronized, we pull out the key from our node by entering the command
```python
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

## BACKUP
🟢 save the located keys in `$HOME/.tangle/node-key` and `$HOME/.tangle/data/validator/YOURNAME/chains/tangle-testnet/keystore/`

## Creating a validator
- Go to the [website](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.tangle.tools#/accounts) and first create a wallet
- We create a validator. To do this, select `Network - Staking - Accounts - Validator`

`logs`
```python
journalctl -fu tangle -o cat
```
`restart`
```
systemctl restart tangle && journalctl -u tangle -f -o cat
```
`delete node`
```
systemctl stop tangle &&
systemctl disable tangle &&
rm /etc/systemd/system/tangle.service &&
systemctl daemon-reload &&
cd
rm -r .tangle
```
