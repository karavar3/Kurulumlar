# Shardeum Testnet Node Guide

![FpgmRBdWcAEc0y2](https://user-images.githubusercontent.com/82613690/221593689-fbe0c9f6-875b-4128-a7ee-e2a24da52b6b.jpg)

## 1. Gereksinimler
#### Official 
- 4 CPU
- 8 GB RAM
- 200 GB SSD


## 2. Gerekli kütüphanelerin kurulumu
```
sudo apt update && sudo apt upgrade -y
sudo apt-get install curl
```
```
sudo apt install docker.io -y
```
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
## 3. Shardeum Validator'ı Kurmak
Sırasıyla "Y","Y","şifre girin" bunlardan sonralarına "enter" diyerek geçebilirsiniz.
```
curl -O https://gitlab.com/shardeum/validator/dashboard/-/raw/main/installer.sh && chmod +x installer.sh && ./installer.sh
```
Bu çıktıyı aldıktan sonra devam edelim

![216381866-e840b39c-1e87-4403-a0d2-32289e206ad4](https://user-images.githubusercontent.com/82613690/221594042-8a6d8420-680e-4e00-8847-2c1115653de0.png)

```
cd
cd .shardeum
./shell.sh
operator-cli gui start

```

## 4. Kendi bilgisayarınızda bir tarayıcı açın
localhost kısmına kendi IP adresini girip tarayıcınıza yapıştırın.
```
https://localhost:8080/ 
```
Yapıştırdıktan sonra tarayıcıda hata alabilirsiniz. Bunun için gelişmiş ayarlardan izin verip, siteye git diyin. Şifrenizi 
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
