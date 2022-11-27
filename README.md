<p align="center">
  <a href="https://orbisworlds.org">
    <img alt="Hero" src="https://user-images.githubusercontent.com/73176377/204139266-5dda062f-3305-49d6-97c2-cc3661aa01f4.jpg" width="100%" />
  </a>
</p>

# Tavsiye Edilen Sistem Gereksinimleri
- CPU: 4 Cores
- RAM: 32 GB
- SSD: 1 TB
- İşletim Sistemi: Ubuntu 20.04LTS

## Güncellemeler
```
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && sudo apt install make clang pkg-config libssl-dev build-essential git jq llvm libudev-dev -y
```

## Go Yükle
```
wget https://go.dev/dl/go1.19.linux-amd64.tar.gz \
&& sudo tar -xvf go1.19.linux-amd64.tar.gz \
&& sudo mv go /usr/local \
&& echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile \
&& source ~/.bash_profile \
&& go version
rm go1.19.linux-amd64.tar.gz
```

## Binary Yükle
```
git clone -b v0.1.1 https://github.com/neutron-org/neutron.git
cd neutron
make install
```

## Versiyon Kontrol
- ```neutrond version --long``` yazdığımızda aşağıdaki gibi sonuç vermeli.
```
name: neutron
server_name: neutrond
version: 0.1.1
commit: a9e8ba5ebb9230bec97a4f2826d75a4e0e6130d9
```

## Init İşlemi
```
neutrond init "<moniker-name>" --chain-id quark-1
```

## Diğer İşlemler

- Aşağıdaki kodları sırasıyla girdikten sonra Node başlatıp sisteme eşitlenmesini bekliyoruz. Daha öncesinde cüzdan oluşturarak Discord faucet https://discord.gg/bg4mKxnVD7 kanalından test tokenı alıyoruz. Şuradan https://neutron.explorers.guru/ ağın blok yüksekliğini öğrenebilirsiniz.
```
curl -s https://raw.githubusercontent.com/neutron-org/testnets/main/quark/genesis.json > ~/.neutrond/config/genesis.json
```
```
sed -i 's|^timeout_commit *=.*|timeout_commit = "2s"|' $HOME/.neutrond/config/config.toml
sed -i 's|^timeout_commit *=.*|minimum-gas-prices = "0.001untrn"|' $HOME/.neutrond/config/app.toml

```
- Seed ve Peer Eklemek
```
seeds="e2c07e8e6e808fb36cca0fc580e31216772841df@seed-1.quark.ntrn.info:26656,c89b8316f006075ad6ae37349220dd56796b92fa@tenderseed.ccvalidators.com:29001"
peers="fcde59cbba742b86de260730d54daa60467c91a5@23.109.158.180:26656,5bdc67a5d5219aeda3c743e04fdcd72dcb150ba3@65.109.31.114:2480,3e9656706c94ae8b11596e53656c80cf092abe5d@65.21.250.197:46656,9cb73281f6774e42176905e548c134fc45bbe579@162.55.134.54:26656,27b07238cf2ea76acabd5d84d396d447d72aa01b@65.109.54.15:51656,f10c2cb08f82225a7ef2367709e8ac427d61d1b5@57.128.144.247:26656,20b4f9207cdc9d0310399f848f057621f7251846@222.106.187.13:40006,5019864f233cee00f3a6974d9ccaac65caa83807@162.19.31.150:55256,2144ce0e9e08b2a30c132fbde52101b753df788d@194.163.168.99:26656,b37326e3acd60d4e0ea2e3223d00633605fb4f79@nebula.p2p.org:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.neutrond/config/config.toml
```
```
sed -i 's|pruning = "default"|pruning = "custom"|g' $HOME/.neutrond/config/app.toml
sed -i 's|pruning-keep-recent = "0"|pruning-keep-recent = "100"|g' $HOME/.neutrond/config/app.toml
sed -i 's|pruning-interval = "0"|pruning-interval = "13"|g' $HOME/.neutrond/config/app.toml
```

```
sudo tee /etc/systemd/system/neutrond.service > /dev/null << EOF
[Unit]
Description=Neutron Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which neutrond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl enable neutrond
sudo systemctl start neutrond
```
- Log Kontrol
```
journalctl -u neutrond -f
```

- Cüzdan Oluşturma
```
neutrond keys add <key-name> 
```

- Validator Oluşturma
```
$ neutrond tx staking create-validator \
--amount 1000000untrn \
--chain-id=quark-1 \
--pubkey=$(neutrond tendermint show-validator) \
--moniker="<moniker-name>" \
--website=<your-node-website> \
--details=<your-node-details> \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--from <key-name> 
```


