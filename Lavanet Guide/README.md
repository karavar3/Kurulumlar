# Lava Testnet Node Guide

![FplcCfVacAEhyJm](https://user-images.githubusercontent.com/82613690/236672646-2e8afdf0-c12a-4c32-829c-0342611a25f8.jpg)


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
sudo apt install -y curl git jq lz4 build-essential unzip
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
NODE_MONIKER="İSİM"
```

```
cd || return
rm -rf lava
git clone https://github.com/lavanet/lava
cd lava || return
git checkout v0.10.1
make install
lavad version
```

```
lavad config keyring-backend test
lavad config chain-id lava-testnet-1
lavad init "$NODE_MONIKER" --chain-id lava-testnet-1
```

```
curl -s https://raw.githubusercontent.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T/main/testnet-1/genesis_json/genesis.json > $HOME/.lava/config/genesis.json
curl -s https://snapshots1-testnet.nodejumper.io/lava-testnet/addrbook.json > $HOME/.lava/config/addrbook.json
```

## 5. Node Configuration
```
SEEDS="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.lava/config/config.toml

sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.lava/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.lava/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.lava/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.lava/config/app.toml

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025ulava"|g' $HOME/.lava/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.lava/config/config.toml
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

## 8. Snapshot

```
sudo systemctl stop lavad

cp $HOME/.lava/data/priv_validator_state.json $HOME/.lava/priv_validator_state.json.backup 

lavad tendermint unsafe-reset-all --home $HOME/.lava --keep-addr-book 
curl https://snapshots1-testnet.nodejumper.io/lava-testnet/lava-testnet-1_2023-05-07.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.lava

mv $HOME/.lava/priv_validator_state.json.backup $HOME/.lava/data/priv_validator_state.json 

sudo systemctl restart lavad
sudo journalctl -u lavad -f --no-hostname -o cat

```

Node senkronize olana kadar bekleyin. Node senkronize olduğunda "False" çıktısı almalısınız.
```
lavad status 2>&1 | jq .SyncInfo
```


## 9. Cüzdan oluşturma ve Token İstemek

Memoniclerinizi kaydetmeyi unutmayın!
```
lavad keys add wallet
```

Test token için [Discord](https://discord.gg/BBgprSw2vn).

## 10. Validatör oluşturma

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

## 11. Update

```
sudo systemctl stop lavad

cd || return
rm -rf lava
git clone https://github.com/lavanet/lava
cd lava || return
git checkout v0.10.1
make install
lavad version

sudo systemctl start lavad
sudo journalctl -u lavad -f --no-hostname -o cat

```


## 12. Node silme
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

