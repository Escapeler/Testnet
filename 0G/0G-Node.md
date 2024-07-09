#0G-Chain Node Deployment

### 1. install dependencies for building from source

```bash
 sudo apt update && \
 sudo apt install curl git jq build-essential gcc unzip wget lz4 openssl -y
```

### 2. install go

```bash
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### 3. set variabel

EDIT YOUR MONIKER

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="<your-moniker>"" >> $HOME/.bash_profile
echo "export OG_CHAIN_ID="zgtendermint_16600-2"" >> $HOME/.bash_profile
echo "export OG_PORT="26"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### 4. download binary

```bash
cd $HOME
rm -rf 0g-chain
git clone -b v0.2.3 https://github.com/0glabs/0g-chain.git
cd 0g-chain
make install
```

### 5. config and init app

```bash
0gchaind config node tcp://localhost:${OG_PORT}657
0gchaind config keyring-backend os
0gchaind config chain-id zgtendermint_16600-2
0gchaind init "test" --chain-id zgtendermint_16600-2
```

### 6. download genesis.json

```bash
wget https://github.com/0glabs/0g-chain/releases/download/v0.2.3/genesis.json -O $HOME/.0gchain/config/genesis.json
```

### 7. Add seeds to the config.toml

```bash
SEEDS="81987895a11f6689ada254c6b57932ab7ed909b6@54.241.167.190:26656,010fb4de28667725a4fef26cdc7f9452cc34b16d@54.176.175.48:26656,e9b4bc203197b62cc7e6a80a64742e752f4210d5@54.193.250.204:26656,68b9145889e7576b652ca68d985826abd46ad660@18.166.164.232:26656" && \
sed -i.bak -e "s/^seeds *=.*/seeds = \"${SEEDS}\"/" $HOME/.0gchain/config/config.toml
```

### 8. Add peers to the config.toml

```bash
peers=""
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.0gchain/config/config.toml
```

### 9. set custom ports in app.toml

```bash
sed -i.bak -e "s%:1317%:${OG_PORT}317%g;
s%:8080%:${OG_PORT}080%g;
s%:9090%:${OG_PORT}090%g;
s%:9091%:${OG_PORT}091%g;
s%:8545%:${OG_PORT}545%g;
s%:8546%:${OG_PORT}546%g;
s%:6065%:${OG_PORT}065%g" $HOME/.0gchain/config/app.toml
```

### 10. set custom ports in config.toml file

```bash
sed -i.bak -e "s%:26658%:${OG_PORT}658%g;
s%:26657%:${OG_PORT}657%g;
s%:6060%:${OG_PORT}060%g;
s%:26656%:${OG_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OG_PORT}656\"%;
s%:26660%:${OG_PORT}660%g" $HOME/.0gchain/config/config.toml
```

### 11. config pruning

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.0gchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.0gchain/config/app.toml
```

### 12. set minimum gas price, enable prometheus and disable indexing

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0ua0gi"|g' $HOME/.0gchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.0gchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.0gchain/config/config.toml
```

### 13. create service file

```bash
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0G node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.0gchain
ExecStart=$(which 0gchaind) start --home $HOME/.0gchain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### 14. enable and start service

```bash
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind
sudo systemctl restart 0gchaind && sudo journalctl -u 0gchaind -f
```
