# Babylon Kurulum Rehberi

![babylon](https://user-images.githubusercontent.com/111747226/223374946-c71aa47c-a81b-4482-a71b-149df576898e.png)



## Sistem gereksinimleri:

- **Ubuntu 20.04+**

NODE TİPİ | CPU     | RAM      | SSD     |
| ------------- | ------------- | ------------- | -------- |
| Babylon | 4          | 8         | 160  |
  
  

- **Gerekli Güncellemeler ve Kurulum**

```
sudo apt update && sudo apt upgrade -y && sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

- **Go Kurulum**

```
ver="1.19.5"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
```

- **Go Versiyon Kontrol**

```
go version
```
- **Binary Yüklemesi**

```
cd $HOME
rm -rf babylon
cd babylon
git checkout v0.19.2
make build
```

- **Cosmovisor**

```
mkdir -p $HOME/.babylond/cosmovisor/genesis/bin
mv build/nibid $HOME/.babylond/cosmovisor/genesis/bin/
rm -rf build
```
```
sudo ln -s $HOME/.babylond/cosmovisor/genesis $HOME/.babylond/cosmovisor/current
sudo ln -s $HOME/.babylond/cosmovisor/current/bin/babylond /usr/local/bin/babylond
```

- **babylond'ün başarılı yüklendiğinin kontrolü**

```
babylond version
```


- **Cosmovisor Kurulumu**

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
```

- **Yapılandırma**

```
mkdir -p ~/.babylond
mkdir -p ~/.babylond/cosmovisor
mkdir -p ~/.babylond/cosmovisor/genesis
mkdir -p ~/.babylond/cosmovisor/genesis/bin
mkdir -p ~/.babylond/cosmovisor/upgrades
```
```
cp $GOPATH/bin/babylond ~/.babylond/cosmovisor/genesis/bin/babylond
```

- **Node Kurulumu**

$NODENAME-isminiz olan yeri değiştirin**
  
```
babylond init $NODENAME --chain-id bbn-test1
```

- **Genesis**

```
wget https://github.com/babylonchain/networks/raw/main/bbn-test1/genesis.tar.bz2
```
```
tar -xjf genesis.tar.bz2 && rm genesis.tar.bz2
```
```
mv genesis.json ~/.babylond/config/genesis.json
```
- **Servis Dosyası Oluşturma**

```
sudo tee /etc/systemd/system/nibid.service > /dev/null << EOF
[Unit]
Description=nibiru-testnet node service
After=network-online.target

[Service]
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=Babylon daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants
Restart=always
RestartSec=3
LimitNOFILE=infinity

Environment="DAEMON_NAME=babylond"
Environment="DAEMON_HOME=${HOME}/.babylond"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"

[Install]
WantedBy=multi-user.target
EOF
```
- **Servis etkinleştirme ve Node başlatma**

```
sudo -S systemctl daemon-reload
sudo -S systemctl enable babylond
sudo -S systemctl start babylond
sudo journalctl -fu babylond -o cat
```

- **Cüzdan Oluşturma**
- cüzdanadi yerine istediğiniz bir adı yazabilirsiniz.
```
babylond keys add cüzdanadi
```

- **Önceden kullandığınız bir cüzdanı eklemek için**
- cüzdanadi yerine istediğiniz bir adı yazabilirsiniz.
```
nibid keys add cüzdanadi --recover
```


- **Babylon Discord Kanalından Faucet**

- **Discord** - **(https://discord.gg/babylonchain)**
```
$request <cüzdan-adresiniz>
```

**Validatör Kurulumu**
> "moniker-adiniz" " dahil silip moniker adınızı yazın, cuzdanadi kısmını silip cüzdanadınızı yazmayı unutmayın.

```
babylond tx checkpointing create-validator \
  --amount="$AMOUNT" \
  --pubkey=$(babylond tendermint show-validator) \
  --moniker=Monikeradi \
  --website="https://coinhunterstr.com/" \
  --details="https://linktr.ee/coinhunters" \
  --chain-id=$CHAIN_ID \
  --gas="auto" \
  --gas-adjustment=1.2 \
  --gas-prices="0.0025ubbn" \
  --keyring-backend=test \
  --from=cüzdanadi \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1"
-y
```

## Nibiru Testnet Explorer [Buradan](https://babylon.explorers.guru/validators) ulaşabilirsiniz.


