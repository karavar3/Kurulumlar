# Lava Testnet Node Guide

![alt text](https://i.hizliresim.com/e5qbrvl.png)

## 1. Gereksinimler
#### Official 
- 4 CPU
- 8 GB RAM
- 100 GB SSD


## 2. Sunucu Hazırlığı
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install make clang pkg-config libssl-dev libclang-dev build-essential git curl ntp jq llvm tmux htop screen unzip cmake -y
```
```
CHAIN_ID=lava-testnet-1
echo "export CHAIN_ID=${CHAIN_ID}" >> $HOME/.profile
source $HOME/.profile
```
## 3. Go Yükleme
```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm go1.19.3.linux-amd64.tar.gz
echo "export PATH=\$PATH:/usr/local/go/bin" >>~/.profile
echo "export PATH=\$PATH:\$(go env GOPATH)/bin" >>~/.profile
source ~/.profile
```
## 4. Node Yükleme
```
git clone https://github.com/lavanet/lava
cd lava
git checkout v0.6.0 
make install
```

```
cd ~
lavad init <your_node_name_here> --chain-id=$CHAIN_ID
```
```
git clone https://github.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T.git
mv GHFkqmTzpdNLDd6T/testnet-1/genesis_json/genesis.json .lava/config
```

## 5. Node Configuration
```
sed -i 's|seeds =.*|seeds = "3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"|g' $HOME/.lava/config/config.toml
sed -i 's/create_empty_blocks = .*/create_empty_blocks = true/g' ~/.lava/config/config.toml
sed -i 's/create_empty_blocks_interval = ".*s"/create_empty_blocks_interval = "60s"/g' ~/.lava/config/config.toml
sed -i 's/timeout_propose = ".*s"/timeout_propose = "60s"/g' ~/.lava/config/config.toml
sed -i 's/timeout_commit = ".*s"/timeout_commit = "60s"/g' ~/.lava/config/config.toml
sed -i 's/timeout_broadcast_tx_commit = ".*s"/timeout_broadcast_tx_commit = "601s"/g' ~/.lava/config/config.toml
```

## 6. Systemed Oluşturma
```
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=Lava Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=always
RestartSec=180
LimitNOFILE=infinity
LimitNPROC=infinity
[Install]
WantedBy=multi-user.target
EOF
```
                                                        
## 7. Start Synchronization
```
sudo systemctl daemon-reload
sudo systemctl start lavad
sudo systemctl enable lavad
sudo journalctl -u lavad -f -n 100
```
Node senkronize olana kadar bekleyin. Node senkronize olduğunda "False" çıktısı almalısınız.
```
lavad status 2>&1 | jq .SyncInfo
```


## 8. Cüzdan oluşturma ve Token İstemek
Memoniclerinizi kaydetmeyi unutmayın!
```
lavad keys add wallet
```
Test token için [Discord](https://discord.gg/BBgprSw2vn).
## 9. Validatör oluşturma
```
lavad tx staking create-validator \
    --amount="10000ulava" \
    --pubkey=$(lavad tendermint show-validator --home "$HOME/.lava/") \
    --moniker="Your_Validator_Name" \
    --chain-id=lava-testnet-1 \
    --commission-rate="0.10" \
    --commission-max-rate="0.20" \
    --commission-max-change-rate="0.01" \
    --min-self-delegation="10000" \
    --gas="auto" \
    --gas-adjustment "1.5" \
    --gas-prices="0.05ulava" \
    --home "$HOME/.lava/" \
    --from=wallet
```


## 10. Node silme
```
sudo systemctl stop lavad
sudo rm -rf .lava
sudo rm -rf lava
sudo rm -rf /go/bin/lavad
sudo rm -rf /etc/systemd/system/lavad.service
sudo rm -rf GHFkqmTzpdNLDd6T
```


👉[Official guide](https://docs.lavanet.xyz/)

👉[Lava Explorer](https://lava.explorers.guru/)

