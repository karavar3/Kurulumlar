# Taiko Testnet Node Guide
![download](https://github.com/karavar3/Kurulumlar/assets/82613690/04b2c2e4-1f96-4757-9976-59cd03d0203e)

## 1. Gereksinimler
#### Official 
- 2 CPU
- 4 GB RAM
- 50 GB SSD


## 2. Gerekli kütüphanelerin kurulumu

Screen oluşturma
```
screen -S anasayfa
```

```
sudo apt update && sudo apt upgrade -y
sudo apt-get install curl
```

```
sudo apt install docker-compose -y
```

```
sudo apt-get update && sudo apt install jq && sudo apt install apt-transport-https ca-certificates curl software-properties-common -y && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - && sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" && sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin && sudo apt-get install docker-compose-plugin
```

## 3. Taiko kurulum
```
git clone https://github.com/taikoxyz/simple-taiko-node.git
```

## 4. Alchemy hesabından taiko için bir dApp oluşturalım
Oluşturacağımız dApp, Ethereum - Sepolia zinciri olacak.

![t1](https://github.com/karavar3/Kurulumlar/assets/82613690/121f7e35-a6bd-4106-aa40-64988710533c)

![Ekran Alıntısı](https://github.com/karavar3/Kurulumlar/assets/82613690/324ed25c-4d29-4a3f-b863-1ced533c4d06)

## 5. App oluşturduktan sonra View key kısmından key bilgilerini alalım

![t2](https://github.com/karavar3/Kurulumlar/assets/82613690/63341f54-c87c-41d6-9056-add8cbba7574)


## 6. Düğümünüzü yapılandırın

```
cd simple-taiko-node
cp .env.sample .env
nano .env
```
                                                        
L1_ENDPOINT_HTTP= Alchemyden aldığınız HTTPS adresini yazıyorsunuz

L1_ENDPOINT_WS= Alchemyden aldığınız WEBSOCKETS adresini yazıyorsunuz

L1_PROVER_PRIVATE_KEY= Bu kısıma Metamask PRİVATE keyinizi yapıştırıyorsunuz

CTRL X + Y + enter ile çıkıyoruz.

![ts](https://github.com/karavar3/Kurulumlar/assets/82613690/b3b59794-53f7-4978-b65e-9299a6ed8341)

## 7. Node Çalıştır

 👉[Faucet Linki] https://sepoliafaucet.com/
 
 Node çalıştır
 ```
 docker compose up -d
 ```
## 8. Grafana Node kontrol

IP adresini yazarak google yapıştır

http://KENDIPADRESİN:3000/d/L2ExecutionEngine/l2-execution-engine-overview


![image](https://github.com/karavar3/Kurulumlar/assets/82613690/3640cb35-8249-4215-b14d-a524a45025b5)


# Kontrol kodları

## 1. Node log görüntüleme
 ```
docker compose logs -f
 ```
 
## 2. Node güncelleme

```
docker compose pull
```



👉[Official guide](https://taiko.xyz/docs/guides/run-a-node) 
