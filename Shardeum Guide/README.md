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
Yapıştırdıktan sonra tarayıcıda hata alabilirsiniz. Bunun için gelişmişe tıkalyıp siteye ilerle diyin. Şifrenizi girdikten sonra

<img width="1440" alt="216432965-714c474d-a742-4032-b6ca-bea7972962e1" src="https://user-images.githubusercontent.com/82613690/221599737-385b0e27-3122-4fc5-85de-12c36bd0f19b.png">

start node diyerek node çalıştırın!

## 5. Faucet 
Linkinden token istemeden önce yukarıdaki görselde yer alan yere metamask adresinizi bağlayın bundan sonra metamask adresinize token isteyebilirsiniz
https://chaindrop.org/?chainid=8082&token=0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee 

## 6. Stake etme

tokenlar geldikten sonra add stake diyerek stake edelim.
                                                        
## 7. Terminalinize geri dönün ve komutları girin
```
operator-cli start
```
Düğümünüzü aşağıdaki komutla izleyebilirsiniz

```
pm2 list
```
![111](https://user-images.githubusercontent.com/82613690/221606938-2663992c-992b-4fd1-81ae-d04a2f9d11d9.PNG)




👉[Official guide](https://docs.shardeum.org/node/run/validator)
