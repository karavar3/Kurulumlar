# Namada Testnet Node Guide
![opengraph](https://user-images.githubusercontent.com/82613690/225137742-6d599592-9773-45c0-b9b5-09240b082d40.jpg)

## 1. Gereksinimler
- Validator	32 GB	500GB-2TB*
- Full Node	8GB DDR4	2TB
- Light Node	TBD	TBD

## 2. Sunucu Hazırlığı
```
Screen -S Namada
```
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config git make libssl-dev libclang-dev libclang-12-dev -y
sudo apt install jq build-essential bsdmainutils ncdu gcc git-core chrony liblz4-tool -y
sudo apt install original-awk uidmap dbus-user-session protobuf-compiler unzip -y
```

## 3. Rust & Go & Protobuf yükleme

```
sudo apt update
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
. $HOME/.cargo/env
curl https://deb.nodesource.com/setup_18.x | sudo bash
sudo apt install cargo nodejs -y < "/dev/null"

cargo --version
```

```
if ! [ -x "$(command -v go)" ]; then
ver="1.20.5"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
fi

go version
```

```
cd $HOME && rustup update
PROTOC_ZIP=protoc-23.3-linux-x86_64.zip
curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v23.3/$PROTOC_ZIP
sudo unzip -o $PROTOC_ZIP -d /usr/local bin/protoc
sudo unzip -o $PROTOC_ZIP -d /usr/local 'include/*'
rm -f $PROTOC_ZIP

protoc --version
```

## 4. Ayarlar

```
sed -i '/public-testnet/d' "$HOME/.bash_profile"
sed -i '/NAMADA_TAG/d' "$HOME/.bash_profile"
sed -i '/WALLET_ADDRESS/d' "$HOME/.bash_profile"
sed -i '/CBFT/d' "$HOME/.bash_profile"
```

```
echo "export NAMADA_TAG=v0.23.2" >> ~/.bash_profile
echo "export CBFT=v0.37.2" >> ~/.bash_profile
echo "export NAMADA_CHAIN_ID=public-testnet-14.5d79b6958580" >> ~/.bash_profile
echo "export BASE_DIR=$HOME/.local/share/namada" >> ~/.bash_profile
```

## 5. bir kullanıcı hesabı oluşturun
```
echo "export WALLET=wallet" >> ~/.bash_profile
```
Not: VALIDATORADI YAZILAN KISIMI DEĞİŞTİR!!
```
echo "export BASE_DIR=$HOME/.local/share/namada" >> ~/.bash_profile
echo "export VALIDATOR_ALIAS=VALIDATORADI" >> ~/.bash_profile

source ~/.bash_profile
```

## 6. NAMADA Yükle

```
cd $HOME && git clone https://github.com/anoma/namada && cd namada && git checkout $NAMADA_TAG
make build-release
cargo fix --lib -p namada_apps

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
cp "$HOME/namada/target/release/namadaw" /usr/local/bin/namadaw && \
cp "$HOME/namada/target/release/namadar" /usr/local/bin/namadar

```

```
cometbft version
namada --version
```

## 7. Systemd Oluşturma

```
sudo tee /etc/systemd/system/namadad.service > /dev/null <<EOF
[Unit]
Description=namada
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.local/share/namada
Environment=TM_LOG_LEVEL=p2p:none,pex:error
Environment=NAMADA_CMT_STDOUT=true
ExecStart=/usr/local/bin/namada node ledger run 
StandardOutput=syslog
StandardError=syslog
Restart=always
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable namadad

```

## 8. PR Oluşturmak için

PUBLIC_IP Kısmına sunucu IP adresinizi $ALIAS kısmına validator adınızı gireceksiniz.

```
namada client utils init-genesis-validator --alias $ALIAS \
--max-commission-rate-change 0.01 --commission-rate 0.05 \
--net-address $PUBLIC_IP:26656
```
Toml dosyasının çıktısını alın
```
cat $HOME/.local/share/namada/pre-genesis/$ALIAS/validator.toml
```
Aldığınız Gentx dosyasını burada Çekme isteği oluşturun
https://github.com/anoma/namada-testnets/pulls


 # Ek olarak
 
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
