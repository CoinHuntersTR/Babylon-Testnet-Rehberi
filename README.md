# Babylon Kurulum Rehberi

![babylon](https://user-images.githubusercontent.com/111747226/223374946-c71aa47c-a81b-4482-a71b-149df576898e.png)



## Sistem gereksinimleri:

- **Ubuntu 20.04+**

NODE TİPİ | CPU     | RAM      | SSD     |
| ------------- | ------------- | ------------- | -------- |
| Babylon | 4          | 8         | 160  |
  
  

- **Gerekli Güncellemeler ve Kurulum**

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install -y curl build-essential git wget jq make gcc tmux chrony lz4 unzip
```
- **Go Kurulum**

```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.19.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
rm go1.19.3.linux-amd64.tar.gz

echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile

```

- **Binary Yüklemesi**

```
cd $HOME && rm -rf babylon
git clone https://github.com/babylonchain/babylon.git
cd babylon
git checkout v0.5.0
make install

```

- **Servisleri Oluşturuyoruz.**

```
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=Babylon Network Node
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which babylond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable babylond

```

- **MONİKER adımızı giriyoruz.**
- " içinde istediğiniz ismi arasında boşluk kalmadan giriyoruz.
```
NODENAME="CoinHunters"
```


- **Config Dosyasını Ayarlıyoruz**

```
BABYLON_PORT=30
echo "export BABYLON_CHAIN_ID=bbn-test1" >> $HOME/.bash_profile
echo "export BABYLON_PORT=${BABYLON_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile

babylond config chain-id $BABYLON_CHAIN_ID
babylond config keyring-backend test
babylond config node tcp://localhost:${BABYLON_PORT}657
babylond init $NODENAME --chain-id $BABYLON_CHAIN_ID
```

- **Genesis ve Addrbook ayarlıyoruz.**

```
wget -qO $HOME/.babylond/config/genesis.json wget "https://snapshot.yeksin.net/babylon/genesis.json"
wget -qO $HOME/.babylond/config/addrbook.json wget "https://snapshot.yeksin.net/babylon/addrbook.json"
```
- **Seed ve Peer ayarlıyoruz.**

```
SEEDS="03ce5e1b5be3c9a81517d415f65378943996c864@18.207.168.204:26656,a5fabac19c732bf7d814cf22e7ffc23113dc9606@34.238.169.221:26656"
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.babylond/config/config.toml
```
- **Config Pruning ayarlıyoruz.**
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.babylond/config/app.toml

sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ubbn\"/" $HOME/.babylond/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.babylond/config/config.toml
sed -i 's|^checkpoint-tag *=.*|checkpoint-tag = "bbn0"|g' $HOME/.babylond/config/app.toml
```
- **Porları ve diğer  ayarları yapıyoruz.**
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${BABYLON_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${BABYLON_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${BABYLON_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${BABYLON_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${BABYLON_PORT}660\"%" $HOME/.babylond/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${BABYLON_PORT}317\"%; s%^address = \":8080\"%address = \":${BABYLON_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${BABYLON_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${BABYLON_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${BABYLON_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${BABYLON_PORT}546\"%" $HOME/.babylond/config/app.toml
```
- **Snapshot Dosyasını yüklüyoruz.**
```
curl -L https://snapshot.yeksin.net/babylon/data.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.babylond
```
- **Servisi başlatıyoruz.**
```
sudo systemctl start babylond
```
- **Log kayıtlarını görüntüleme**
```
sudo journalctl -u babylond -f -o cat
```
- **Node kurmadan önce True-False durumuna bakma**
- False olana kadar bekliyoruz.
```
babylond status 2>&1 | jq .SyncInfo
```

- **Cüzdan Oluşturma**
- cüzdanadi yerine istediğiniz bir adı yazabilirsiniz.
```
babylond keys add cüzdanadi
```

- **Önceden kullandığınız bir cüzdanı eklemek için**
- cüzdanadi yerine istediğiniz bir adı yazabilirsiniz.
```
babylond keys add cüzdanadi --recover
```


- **Babylon Discord Kanalından Faucet**

- **Discord** [Buradan](https://discord.gg/babylonchain) ulaşabilirsiniz.
```
$request <cüzdan-adresiniz>
```

**Validatör Kurulumu**
> "moniker-adiniz" " dahil silip moniker adınızı yazın, cuzdanadi kısmını silip cüzdanadınızı yazmayı unutmayın.
- Faucetten az miktarda token verdiği için kurulumu yapmadan önce biraz token isteyin. 
```
babylond tx checkpointing create-validator \
  --amount=500ubbn \
  --pubkey=$(babylond tendermint show-validator) \
  --moniker=Monikeradi \
  --details="https://linktr.ee/coinhunters" \
  --website="https://coinhunterstr.com/" \
  --chain-id=bbn-test1 \
  --gas="auto" \
  --gas-adjustment=1.2 \
  --gas-prices="0.0025ubbn" \
  --keyring-backend=test \
  --from=cüzdanadi \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
-y
```

## Babylon Testnet Explorer [Buradan](https://babylon.explorers.guru/validators) ulaşabilirsiniz.


