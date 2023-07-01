# Namada Testnet Node Guide
![opengraph](https://user-images.githubusercontent.com/82613690/225137742-6d599592-9773-45c0-b9b5-09240b082d40.jpg)

## 1. Gereksinimler
- 4 CPU
- 8 GB RAM
- 500 GB SSD

## 2. Sunucu Hazırlığı
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config git make libssl-dev libclang-dev libclang-12-dev -y
sudo apt install jq build-essential bsdmainutils ncdu gcc git-core chrony liblz4-tool -y
sudo apt install uidmap dbus-user-session protobuf-compiler unzip -y

```

## 3. Rust & Go & Protobuf yükleme

```
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
. $HOME/.cargo/env
curl https://deb.nodesource.com/setup_18.x | sudo bash
sudo apt install cargo nodejs -y < "/dev/null"
```

```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.19.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.4.linux-amd64.tar.gz
rm go1.19.4.linux-amd64.tar.gz
echo "export PATH=\$PATH:/usr/local/go/bin" >>~/.profile
echo "export PATH=\$PATH:\$(go env GOPATH)/bin" >>~/.profile
source ~/.profile
go version
```

```
cd $HOME && rustup update
PROTOC_ZIP=protoc-23.3-linux-x86_64.zip
curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v23.3/$PROTOC_ZIP
sudo unzip -o $PROTOC_ZIP -d /usr/local bin/protoc
sudo unzip -o $PROTOC_ZIP -d /usr/local 'include/*'
rm -f $PROTOC_ZIP
```

## 4. /.bash_profile içindeki değişkenleri kontrol edin ve doğru yerde gerçekleştirin

```
sed -i '/public-testnet/d' "$HOME/.bash_profile"
sed -i '/NAMADA_TAG/d' "$HOME/.bash_profile"
sed -i '/WALLET_ADDRESS/d' "$HOME/.bash_profile"
sed -i '/CBFT/d' "$HOME/.bash_profile"
```

## 5. Ayarlar

```
echo "export NAMADA_TAG=v0.17.5" >> ~/.bash_profile
echo "export CBFT=v0.37.2" >> ~/.bash_profile
echo "export CHAIN_ID=public-testnet-10.3718993c3648" >> ~/.bash_profile
```

## 6. bir kullanıcı hesabı oluşturun
```
echo "export WALLET=wallet" >> ~/.bash_profile
```
Not: VALIDATORADI YAZILAN KISIMI DEĞİŞTİR!!
```
echo "export BASE_DIR=$HOME/.local/share/namada" >> ~/.bash_profile
echo "export VALIDATOR_ALIAS=VALIDATORADI" >> ~/.bash_profile 
```
```
source ~/.bash_profile
```

## 7. NAMADA Yükle
```
cd $HOME && git clone https://github.com/anoma/namada && cd namada && git checkout $NAMADA_TAG
make build-release
```
```
cd $HOME && git clone https://github.com/cometbft/cometbft.git && cd cometbft && git checkout $CBFT
make build
```
```
cd $HOME && cp $HOME/cometbft/build/cometbft /usr/local/bin/cometbft && \
cp "$HOME/namada/target/release/namada" /usr/local/bin/namada && \
cp "$HOME/namada/target/release/namadac" /usr/local/bin/namadac && \
cp "$HOME/namada/target/release/namadan" /usr/local/bin/namadan && \
cp "$HOME/namada/target/release/namadaw" /usr/local/bin/namadaw

```

```
cometbft version
namada --version
```

## 8. Ağa katıl
```
cd $HOME && namada client utils join-network --chain-id $CHAIN_ID
sudo journalctl -u namadad -f -o cat 
```

## 9. Systemd Hizmetini Kurun
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

## 10. Hizmeti etkinleştirin
```
sudo systemctl daemon-reload
sudo systemctl enable namadad
sudo systemctl start namadad
```

## 11. Senkronize olana kadar bekleyin
Senkronize olduğunda "catching_up": false olması lazım
```
curl -s localhost:26657/status
```

## 12. Doğrulayıcı Hesabı Başlatın
```
cd $HOME
namada wallet address gen --alias $WALLET --unsafe-dont-encrypt
```
```
cd $HOME
namada client transfer \
  --source faucet \
  --target $WALLET \
  --token NAM \
  --amount 1000 \
  --signer $WALLET
   
 ```

```
cd $HOME
namada client init-validator \
--alias $VALIDATOR_ALIAS \
--source $WALLET \
--commission-rate 0.05 \
--max-commission-rate-change 0.01 \
--signer $WALLET \
--gas-amount 100000000 \
--gas-token NAM \
--unsafe-dont-encrypt
```

## 13. Faucet

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
 
 ## 14. Bakiyeni kontrol et
 ```
 namada client balance --owner $VALIDATOR_ALIAS --token NAM
 ```
 ## 15. Stake işlemi
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
 
👉[Official guide](https://docs.namada.net/introduction/testnets)
