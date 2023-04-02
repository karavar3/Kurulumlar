# Nibiru Testnet Node Guide

![download](https://user-images.githubusercontent.com/82613690/221673641-4b932198-518c-4984-8837-6d79402b5c39.jpg)



## 1. Gereksinimler
#### Official 
- 4 CPU
- 16 GB RAM
- 1000 GB SSD


## 2. Sunucu Hazırlığı
```
sudo apt update && sudo apt upgrade -y

sudo apt install curl git wget htop tmux build-essential jq make gcc -y
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
cd $HOME
git clone https://github.com/NibiruChain/nibiru
cd nibiru
git checkout v0.19.2
make install
nibid version

```


## 5. Zincire bağlan

bir isim koymayı unutmayın!
  
```
nibid init MONİKER-ADINIZ --chain-id=nibiru-itn-1 --home $HOME/.nibid
```

## 6.Genesis 

Genesis'i indir
```
NETWORK=nibiru-itn-1
curl -s https://networks.itn.nibiru.fi/$NETWORK/genesis > $HOME/.nibid/config/genesis.json

```
Seeds Listesini Güncelleyin
```
NETWORK=nibiru-itn-1
sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/seeds)'"|g' $HOME/.nibid/config/config.toml

```
Minimum gaz fiyatlarını belirleyin
```
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml

```
Ağ ile daha hızlı yetişmek için durum eşitleme parametrelerini ayarlayın (isteğe bağlı, ancak önerilir)
```
NETWORK=nibiru-itn-1
sed -i 's|enable =.*|enable = true|g' $HOME/.nibid/config/config.toml
sed -i 's|rpc_servers =.*|rpc_servers = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/rpc_servers)'"|g' $HOME/.nibid/config/config.toml
sed -i 's|trust_height =.*|trust_height = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_height)'"|g' $HOME/.nibid/config/config.toml
sed -i 's|trust_hash =.*|trust_hash = "'$(curl -s https://networks.itn.nibiru.fi/$NETWORK/trust_hash)'"|g' $HOME/.nibid/config/config.toml
```
                                                        
## 7. Systemd Oluşturna
```
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=nibiru
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start --home $HOME/.nibid
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```



## 8. Hizmeti etkinleştirin 

```
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f
```

## 9. Cüzdan Oluşturma
Kayıt olurken kullandığımız cüzdanın mnemoniclerini giriniz!
```
nibid keys add CÜZDAN-ADINIZ --recover
```

## 10. Faucet
Test token için [Discord](https://discord.gg/nibiru)

## 11. Snapshot (isteğe bağlı, ancak önerilir)
```
sudo systemctl stop nibid

cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup 

nibid tendermint unsafe-reset-all --home $HOME/.nibid --keep-addr-book 
curl https://snapshots2-testnet.nodejumper.io/nibiru-testnet/nibiru-itn-1_2023-04-01.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nibid

mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json 

sudo systemctl start nibid
sudo journalctl -u nibid -f --no-hostname -o cat
```

## 12. Validatör oluşturmadan önce kontrol

Senkronizasyon durumunu kontrol edin, düğümünüz tamamen senkronize edildikten sonra, yukarıdan gelen çıktı şunları söyleyecektir:false
```
nibid status 2>&1 | jq .SyncInfo

```
bakiyenizi kontrol edin
```
nibid query bank balances CÜZDAN-ADRESİNİZ
```
Senkronize olduktan ve token aldıktan sonra Validator oluşturalım!

## 13. Validator oluşturma
Girmiş olduğunuz Moniker ve Cüzdan adlarını aşağıda yazınız!

```
nibid tx staking create-validator \
--amount 10000000unibi \
--commission-max-change-rate "0.1" \
--commission-max-rate "0.20" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(nibid tendermint show-validator) \
--moniker NODEADINIZ \
--chain-id nibiru-itn-1 \
--gas-prices 0.025unibi \
--from CÜZDANADINIZ

```

## 14. Log görüntüleme
```
sudo journalctl -u nibid -f -o cat
```

# Pricefeeder Kurulumu

## 1. pricefeeder ikili dosyasını kurun
```
curl -s https://get.nibiru.fi/pricefeeder! | bash

```
## 2. Değişkenleri Ayarlama
Değişkenleri  FEEDER_MNEMONIC ve VALIDATOR_ADDRESS ayarlayın.
```
export CHAIN_ID="nibiru-itn-1"
export GRPC_ENDPOINT="localhost:9090"
export WEBSOCKET_ENDPOINT="ws://localhost:26657/websocket"
export EXCHANGE_SYMBOLS_MAP='{ "bitfinex": { "ubtc:uusd": "tBTCUSD", "ueth:uusd": "tETHUSD", "uusdt:uusd": "tUSTUSD" }, "binance": { "ubtc:uusd": "BTCUSD", "ueth:uusd": "ETHUSD", "uusdt:uusd": "USDTUSD", "uusdc:uusd": "USDCUSD", "uatom:uusd": "ATOMUSD", "ubnb:uusd": "BNBUSD", "uavax:uusd": "AVAXUSD", "usol:uusd": "SOLUSD", "uada:uusd": "ADAUSD", "ubtc:unusd": "BTCUSD", "ueth:unusd": "ETHUSD", "uusdt:unusd": "USDTUSD", "uusdc:unusd": "USDCUSD", "uatom:unusd": "ATOMUSD", "ubnb:unusd": "BNBUSD", "uavax:unusd": "AVAXUSD", "usol:unusd": "SOLUSD", "uada:unusd": "ADAUSD" } }'
export FEEDER_MNEMONIC="CÜZDANKELİMELERİNİZİYAZIN"
export VALIDATOR_ADDRESS="VALOPERADRESİNİZİYAZIN"

```
## 3. Systemd Hizmetini Kurun
```
sudo tee /etc/systemd/system/pricefeeder.service<<EOF
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

[Install]
WantedBy=multi-user.target
EOF

```
## 4. Hizmeti etkinleştirin 

```
sudo systemctl daemon-reload && \
sudo systemctl enable pricefeeder && \
sudo systemctl start pricefeeder
```

## 5. Log görüntüleme
```
sudo journalctl -u pricefeeder -f -o cat
```
# Faydalı Komutlar

## 1. Kendinize ya da başka validatore delege etme
```
nibid tx staking delegate VALOPERADRESİNİZİYAZIN 1000000unibi --from CÜZDANADINIZIYAZIN --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y

```

## 2. Redelege
```
nibid tx staking redelegate gönderenvaloper alıcıvaloperadres 10000000unibi --from CÜZDANADINIZIYAZIN --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y

```

## 3. Cüzdandan cüzdana transfer
```
nibid tx bank send GÖNDERENADRES ALICIADRES 10000000unibi --from CÜZDANADINIZIYAZIN --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y

```

## 4. Biriken Ödülleri toplama
```
nibid tx distribution withdraw-all-rewards --from CÜZDANADINIZIYAZIN --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y

```

## 5. Oy kullanma
```
nibid tx gov vote 1 yes --from CÜZDANADINIZIYAZIN --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y

```
## 6. Unjailden çıkma
```
nibid tx slashing unjail --from CÜZDANADINIZIYAZIN --chain-id nibiru-itn-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025unibi -y
```

## 7. Validator Düzenleme
```
nibid tx staking edit-validator \
--new-moniker YENİADINIZIYAZIN \
--identity KEYBASE.IO ID'NİZ \
--details "AÇIKLAMA" \
--website "WEBSİTEADRESNİZ" \
--chain-id nibiru-itn-1 \
--gas-prices 0.025unibi \
--from CÜZDANADINIZ
```

## 8. Node Silme komutları

```
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibi* -rf
sudo rm $(which nibid) -rf
sudo rm $HOME/.nibid* -rf
sudo rm $HOME/nibiru -rf
sed -i '/NIBIRU_/d' ~/.bash_profile
```


👉[Official guide](https://nibiru.fi/docs/run-nodes/validators/pricefeeder.html#)

👉[Nibiru Explorer](https://testnet.itrocket.net/nibiru/staking)
