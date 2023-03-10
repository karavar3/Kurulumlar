# Aleo Deploy Guide

## 1. Aleo wallet oluşturalım.

https://aleo.tools/ Line giderek “Generate” butonuna basalım.

## 2. Token almak için Tweet atalım
@AleoFaucet send 10 credits to $YOUR_WALLET_ADDRESS

## 3. Kurmuş olduğumuz VPS bağlanarak

### SnarkOS kuralım

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install

screen -S anasayfa

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

git clone https://github.com/AleoHQ/snarkOS.git --depth 1

cd snarkOS
./build_ubuntu.sh
source $HOME/.cargo/env
cargo install --path .

```
### Sonrasında Leo Dilini kuralım

```
cd

git clone https://github.com/AleoHQ/leo
cd leo

cargo install --path .

leo 
```

## 4. Test uygulamanızı dağıtın

```
cd $HOME
mkdir demo_deploy_Leo_app && cd demo_deploy_Leo_app

WALLETADDRESS=""
⚠️ Kaydettiğiniz cüzdan adresinizi $WALLETADDRESS "aleo1… " atayın

APPNAME=helloworld_"${WALLETADDRESS:4:6}"
echo $APPNAME
leo new "${APPNAME}"
cd "${APPNAME}" && leo run && cd -
PATHTOAPP=$(realpath -q $APPNAME)
echo $PATHTOAPP

cd $PATHTOAPP && cd ..

PRIVATEKEY=""
⚠️ Daha önce kaydettiğiniz özel adrese $PRIVATEKEY atayın


⚠️ faucet den gelen linke giderek record (Ciphertext) kaydedin. 
Bundan sonra https://aleo.tools/ linke giderek record kısmında View Key ve
Record (Ciphertext) yerlerine yazınız. Altta size verdiği Record (Plaintext)
kaydederek aşşağıda RECORD="" ("") arasına yazarak konsola yapıştırın.

RECORD=""


snarkos developer deploy "${APPNAME}.aleo" --private-key "${PRIVATEKEY}" --query "https://vm.aleo.org/api" --path "./${APPNAME}/build/" --broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" --fee 600000 --record "${RECORD}"

```

## 4. İşlemler bittikten sonra
işlemler bittikten sonra konsolda ✅ Successfully deployed ‘helloworld_ görmeniz lazım
