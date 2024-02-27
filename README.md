# Self Chain Kurulum Rehberi

![babylon](https://blog.selfchain.xyz/content/images/size/w1200/2024/02/Self-Chain-Incentivized-Testnet-2-01.png)



## Sistem gereksinimleri:

**Ubuntu 22.04+**

NODE TİPİ | CPU     | RAM      | SSD     |
| ------------- | ------------- | ------------- | -------- |
| Self | 4          | 8         | 160  |
  
  

**Gerekli Güncellemeler ve Kurulum**

```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```
**Go Kurulum**

```
rm -rf $HOME/go
sudo rm -rf /usr/local/go
cd $HOME
curl https://dl.google.com/go/go1.20.5.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
go version
```

**Seld Chain indiriyoruz**
```
cd $HOME
mkdir -p $HOME/go/bin/
curl -O http://37.60.236.233/selfchain/selfchaind
chmod +x selfchaind
mv selfchaind $HOME/go/bin/
selfchaind version
```


**MONİKER adımızı giriyoruz.**
> MonikerName yerine istediğiniz ismi arasında boşluk kalmadan giriyoruz.
```
selfchaind init NodeName --chain-id=self-dev-1
```
> Bu komuttan sonra aşağıdaki gibi bir çıktı alacaksınız. Orada Node ID'niz yazıyor bir yere not etmeyi unutmayın!

![Ekran görüntüsü 2024-02-27 162626](https://github.com/CoinHuntersTR/Self-Chain/assets/111747226/153c0847-ee62-42ec-8e04-3c973fbb86df)

**Genesis Dosyalarını indiriyoruz.**
```
curl -Ls https://ss-t.babylon.nodestake.org/genesis.json > $HOME/.babylond/config/genesis.json
```

**Addrbook indiriyoruz.**
```
curl -Ls https://ss-t.babylon.nodestake.org/addrbook.json > $HOME/.babylond/config/addrbook.json
```

**Servis Dosyasını Oluşturuyoruz.**
```
sudo tee /etc/systemd/system/babylond.service > /dev/null <<EOF
[Unit]
Description=babylond Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=$(which babylond) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable babylond
```

**Snapshot (Opsiyonel)**
```
SNAP_NAME=$(curl -s https://ss-t.babylon.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss-t.babylon.nodestake.org/${SNAP_NAME}  | lz4 -c -d - | tar -x -C $HOME/.babylond
```
**Node Tekrar Başlatıyoruz**
```
sudo systemctl restart babylond
```
```
sudo journalctl -u babylond -f --no-hostname -o cat
```


**Node Tekrar Başlatıyoruz**
> Senkronize olup olmadığımızı kontrol etmek için aşağıdaki kodu kullanıyoruz. `false` çıktısı aldığımızda işlem tamamdır. 
```
babylond status 2>&1 | jq .SyncInfo
```

**Cüzdan Oluşturma**
> `Cüzdanismi` yazan yeri silip kendinize bir cüzdan oluşturabilirsiniz.

> Sizden bir şifre belirlemenizi isteyecek (2 kere aynı şifreyi gireceğiz.) unutmayacağınız bir şifre girin!

> Size verilen gizli kelimeleri bir yere not etmeyi unutmayın! 

```
babylond keys add Cüzdanismi
```

> Eğer kullandığınız bir cüzdanı eklemek istiyorsanız. `Cüzdanismi` yazan yeri değiştirmeyi unutmayın!

> Sizden şifre isteyecek (2 kere aynı şifreyi giriyoruz.)
```
babylond keys add Cüzdanismi --recvoer
```

**Cüzdanda Token Değerini Görme**
> `Cüzdanismi` yazan yeri, verdiğiniz cüzdan ismi ile değiştirin. Cüzdanınıza token gelip gelmediğini kontrol etmiş olursunuz.
```
babylond q bank balances $(babylond keys show Cüzdanismi -a)
```

**BLS key oluşturuyoruz.**

> Burada `Cüzdanismi` yazan yere, cüzdana verdiğiniz ismi girin.

> BLS keyinizi bir yere not edersiniz.
```
babylond create-bls-key $(babylond keys show Cüzdanismi -a)
```
```
sudo systemctl restart babylond
```
```
sed -i -e "s|^key-name *=.*|key-name = \"wallet\"|" $HOME/.babylond/config/app.toml
```
```
sed -i -e "s|^timeout_commit *=.*|timeout_commit = \"30s\"|" $HOME/.babylond/config/config.toml
```

**Validator Oluşturuyoruz..**

> `Cüzdanismi` yazan yere kendi cüzdan adınızı giriyoruz.

> `MonikerName` yerine kendi verdiğiniz Moniker adınızı giriyoruz.

> `website` "" içine kendi twitter veya websitesinizi ekleyebilirsiniz.

>  `details` "" içine istediğiniz bir şey yazabilirsiniz. 
```
babylond tx checkpointing create-validator \
--amount=1000000ubbn \
--pubkey=$(babylond tendermint show-validator) \
--moniker=MonikerName \
--details="Coin Hunters Community" \
--website="" \
--chain-id=bbn-test-3 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=Cüzdanismi \
--gas-prices=0.1ubbn \
--gas-adjustment=1.5 \
--gas=auto \
-y
```


**Babylon Explorer**
> Tüm adımları tamamladıktan sonra [BURADAN](https://explorer.nodestake.org/babylon-testnet/staking) kendi validatorünüzü kontrol edebilirsiniz.

