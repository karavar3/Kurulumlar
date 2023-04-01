# Namada Testnet Node Guide
![opengraph](https://user-images.githubusercontent.com/82613690/225137742-6d599592-9773-45c0-b9b5-09240b082d40.jpg)

## 1. Gereksinimler
- 4 CPU
- 8 GB RAM
- 500 GB SSD

## 2. Sunucu Hazırlığı
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev libclang-dev -y
sudo apt install jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
sudo apt install -y uidmap dbus-user-session

```

## 3. Rust ve Go yükleme
```
sudo apt-get update -y
sudo apt-get install build-essential make pkg-config libssl-dev libclang-dev -y
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.19.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.4.linux-amd64.tar.gz
rm go1.19.4.linux-amd64.tar.gz
echo "export PATH=\$PATH:/usr/local/go/bin" >>~/.profile
echo "export PATH=\$PATH:\$(go env GOPATH)/bin" >>~/.profile
source ~/.profile

```
## 4. Ayarlar
```
echo "export NAMADA_TAG=v0.14.3" >> ~/.bash_profile
echo "export TM_HASH=v0.1.4-abciplus" >> ~/.bash_profile
source ~/.bash_profile
```
## 5. NAMADA Yükle
```
git clone https://github.com/anoma/namada && cd namada && git checkout $NAMADA_TAG
make build-release
```
```
git clone https://github.com/heliaxdev/tendermint && cd tendermint && git checkout $TM_HASH
make build
```
```
cd $HOME && cp $HOME/tendermint/build/tendermint  /usr/local/bin/tendermint && cp "$HOME/namada/target/release/namada" /usr/local/bin/namada && cp "$HOME/namada/target/release/namadac" /usr/local/bin/namadac && cp "$HOME/namada/target/release/namadan" /usr/local/bin/namadan && cp "$HOME/namada/target/release/namadaw" /usr/local/bin/namadaw
```

## 5. Node Çalıştır
```
echo "export CHAIN_ID=public-public-testnet-6.0.a0266444b06" >> ~/.bash_profile
cd $HOME && namada client utils join-network --chain-id $CHAIN_ID
```
```
cd $HOME && wget "https://github.com/heliaxdev/anoma-network-config/releases/download/public-testnet-5.0.d25aa64ace6/public-testnet-6.0.a0266444b06.tar.gz"
tar xvzf "$HOME/public-testnet-6.0.a0266444b06.tar.gz"
```

## 6. Systemd Hizmetini Kurun
```
sudo tee /etc/systemd/system/namadad.service > /dev/null <<EOF
[Unit]
Description=namada
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.namada
Environment=NAMADA_LOG=debug
Environment=NAMADA_TM_STDOUT=true
ExecStart=/usr/local/bin/namada --base-dir=$HOME/.namada node ledger run 
StandardOutput=syslog
StandardError=syslog
Restart=always
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## 7. Hizmeti etkinleştirin
```
sudo systemctl daemon-reload
sudo systemctl enable namadad
sudo systemctl start namadad
```
## 8. bir kullanıcı hesabı oluşturun
```
echo "export WALLET=Wallet" >> ~/.bash_profile
namada wallet address gen --alias $WALLET
```
Not: VALIDATORADI YAZILAN KISIMI DEĞİŞTİR!!
```
echo "export VALIDATOR_ALIAS=VALIDATORADI" >> ~/.bash_profile 
```


## 9. Doğrulayıcı Hesabı Başlatın
```
namada client init-validator \
  --alias $VALIDATOR_ALIAS \
  --source $WALLET \
  --commission-rate 0.05 \
  --max-commission-rate-change 0.01
```

## 10. Faucet
```
cd $HOME
namada client transfer \
    --token NAM \
    --amount 1000 \
    --source faucet \
    --target $VALIDATOR_ALIAS \
    --signer $VALIDATOR_ALIAS
   
 ```
 Not: İşlem başına musluktan maksimum 1000 NAM alınabilir, bu nedenle daha fazlasını elde etmek için bunu birden çok kez çalıştırın
 
 ## 11. Bakiyeni kontrol et
 ```
 namada client balance --owner $VALIDATOR_ALIAS --token NAM
 ```
 ## 12. Stake işlemi
 Not: enter-amount yazan kısıma miktarı girin.
 ```
 namada client bond \
  --validator $VALIDATOR_ALIAS \
  --amount <enter-amount>
  ```
  
 # Ek olarak
 Logları Kontrol Etmek için
 ```
 journalctl -u namadad -f -o cat
 ```
  
  Node Silmek
 ```
systemctl stop namadad && systemctl disable namadad
rm /etc/systemd/system/namadad* -rf
rm $(which namadad) -rf
rm $HOME/.namada* -rf
rm $HOME/namada -rf
rm $HOME/tendermint -rf
 ``` 
 
