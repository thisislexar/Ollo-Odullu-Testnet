# Kuruluma başlıyoruz. Öncelike sunucumuza gerekli güncellemeleri ve kurulumları yapalım.

```
sudo su
cd
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

# Go kuruyoruz.

```
cd
ver="1.18.5"
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

# Binary dosyalarını indiriyoruz.

```
cd
git clone https://github.com/OllO-Station/ollo.git
cd ollo
make install
```

# Node'u başlatıyoruz.
```
ollod init <NODEADINIZ> --chain-id ollo-testnet-0
```

# Genesis ve addrbook dosyalarını indiriyoruz.
```
curl https://raw.githubusercontent.com/OllO-Station/ollo/master/networks/ollo-testnet-0/genesis.json | jq .result.genesis > $HOME/.ollo/config/genesis.json
wget -O $HOME/.ollo/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Ollo/addrbook.json"
```

# Seed/peer/minimum gas price gibi ayarlamaları yapıyoruz.
```
SEEDS=""
PEERS="2a8f0fada8b8b71b8154cf30ce44aebea1b5fe3d@145.239.31.245:26656,1173fe561814f1ecb8b8f19d1769b87cd576897f@185.173.157.251:26656,489daf96446f104d822fae34cd4aa7a9b5cebf65@65.21.131.215:26626,f43435894d3ae6382c9cf95c63fec523a2686345@167.235.145.255:26656,2eeb90b696ba9a62a8ad9561f39c1b75473515eb@77.37.176.99:26656,9a3e2725e02d1c420a5d500fa17ce0ef45ddc9e8@65.109.30.117:29656,91f1889f22975294cfbfa0c1661c63150d2b9355@65.108.140.222:30656,d38fcf79871189c2c430473a7e04bd69aeb812c2@78.107.234.44:16656,f795505ac42f18e55e65c02bb7107b08d83ad837@65.109.17.86:37656,6368702dd71e69035dff6f7830eb45b2bae92d53@65.109.57.161:15656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.ollo/config/config.toml
```
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OLLO_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${OLLO_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OLLO_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OLLO_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OLLO_PORT}660\"%" $HOME/.ollo/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OLLO_PORT}317\"%; s%^address = \":8080\"%address = \":${OLLO_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OLLO_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OLLO_PORT}091\"%" $HOME/.ollo/config/app.toml
```
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utollo\"/" $HOME/.ollo/config/app.toml
```
```
ollod tendermint unsafe-reset-all --home $HOME/.ollo
```
# Servis dosyası oluşturuyoruz.
```
sudo tee /etc/systemd/system/ollod.service > /dev/null <<EOF
[Unit]
Description=ollo
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ollod) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```


# Servisi başlatıyoruz.
```
sudo systemctl daemon-reload
sudo systemctl enable ollod
sudo systemctl restart ollod && sudo journalctl -u ollod -f -o cat
```

# StateSync atıyoruz.
```
SNAP_RPC=213.239.217.52:35657
peers="3f54183cf5a712678dc4dff57fa49a5522918727@38.242.130.16:32657,1be12d239ca70a906264de0f09e0ffa01c4211ba@138.201.136.49:26656,06658ccd5c119578fb662633234a2ef154881b94@18.144.61.148:26656,a77c2afc500569a453b7fb64c8a804878dc6e7be@65.108.127.215:26856,2eeb90b696ba9a62a8ad9561f39c1b75473515eb@77.37.176.99:26656,eaee85418b4fc3e7e2e298bb8deb5a8f49956859@5.9.13.234:26856,6e8c603e5eeefd4b83d0575951209c3b495848d6@65.108.69.68:26858,45acf9ea2f2d6a2a4b564ae53ea3004f902d3fb7@185.182.184.200:26656,62b5364abdfb7c0934afaddbd0704acf82127383@65.108.13.185:27060,f599dcd0a09d376f958910982d82351a6f8c178b@95.217.118.96:26878,e2b22ed4b00f37adafed6d711432f612821f5943@77.52.182.194:26656,d38fcf79871189c2c430473a7e04bd69aeb812c2@78.107.234.44:16656,0b4474bc96d72586e1be1860db731522d05fdeef@181.41.142.78:11523,1173fe561814f1ecb8b8f19d1769b87cd576897f@185.173.157.251:26656,489daf96446f104d822fae34cd4aa7a9b5cebf65@65.21.131.215:26626,8559490d439f774e39818c3a8f05a750c6ae6ff8@95.216.151.32:32657,ef56914e54fde621ca71d171c0711166d281b1bc@65.21.149.47:32657,969f3672d9d302c374102aa8ddb1c79672333127@95.216.149.119:32657"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.ollo/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.ollo/config/config.toml
ollod tendermint unsafe-reset-all --home $HOME/.ollo --keep-addr-book
systemctl restart ollod && journalctl -u ollod -f -o cat
```
