# Using Docker

## 1. Install Linux dependencies.
```
sudo apt update \
&& apt-get install -y --no-install-recommends \
apt-transport-https \
ca-certificates \
curl \
software-properties-common
```


## 2. Install Docker.
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt update \
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

raiso? cb ini cak
```
sudo apt update \
sudo apt install docker.io curl -y \
sudo systemctl start docker \
sudo systemctl enable docker \
sudo curl -L https://github.com/docker/compose/releases/download/$VERSION/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose \
sudo chmod +x /usr/local/bin/docker-compose
```

## 3. Add your user to docker Group.
```
sudo usermod -aG docker ${USER}
```


## 4. Install Docker Compose.
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```


## 5. Create SUI directory.
```
mkdir -p $HOME/sui
cd $HOME/sui
```

## 6. Download fullnode.yaml.
```
wget -O $HOME/sui/fullnode-template.yaml https://github.com/MystenLabs/sui/raw/main/crates/sui-config/data/fullnode-template.yaml
```

## 7. Download genesis state file.
```
wget -O $HOME/sui/genesis.blob  https://github.com/MystenLabs/sui-genesis/raw/main/testnet/genesis.blob
```

## 8. Download docker-compose file and update Sui Node image in it:
```
IMAGE="mysten/sui-node:2d07756360c28e35d7c60816bb0f1ed94ccf356e"
wget -O $HOME/sui/docker-compose.yaml https://raw.githubusercontent.com/MystenLabs/sui/main/docker/fullnode/docker-compose.yaml
sed -i.bak "s|image:.*|image: $IMAGE|" $HOME/sui/docker-compose.yaml
```

raiso? pake yg ini cak:
```
wget https://raw.githubusercontent.com/MystenLabs/sui/main/docker/fullnode/docker-compose.yaml
wget https://github.com/MystenLabs/sui/raw/main/crates/sui-config/data/fullnode-template.yaml
wget https://github.com/MystenLabs/sui-genesis/raw/main/testnet/genesis.blob
```

## 9. Start SUI Full Node in the Docker container.
```
docker-compose up -d
```

## 10. Check logs from SUI Node container:
```
docker ps -a
```

**run di screen:**
```
docker logs -f <container id>
```


# Using System Service


## 1. Install Linux dependencies.
```
sudo apt-get update \
&& sudo apt-get install -y --no-install-recommends \
tzdata \
ca-certificates \
build-essential \
libssl-dev \
libclang-dev \
pkg-config \
openssl \
protobuf-compiler \
cmake
```

## 2. Install Rust.
```
sudo curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
rustc --version
```

## 3. Clone GitHub SUI repository.
```
cd $HOME
git clone https://github.com/MystenLabs/sui.git
cd sui
git remote add upstream https://github.com/MystenLabs/sui
git fetch upstream
git checkout -B testnet --track upstream/testnet
```

## 4. Create directory for SUI db and genesis.
```
mkdir $HOME/.sui
```

## 5. Download genesis file (instead of placeholder use the link to genesis you received in email).
```
wget -O $HOME/.sui/genesis.blob  https://github.com/MystenLabs/sui-genesis/raw/main/testnet/genesis.blob
```

## 6. Make a copy of fullnode.yaml and update path to db and genesis file in it.
```
cp $HOME/sui/crates/sui-config/data/fullnode-template.yaml $HOME/.sui/fullnode.yaml
sed -i.bak "s|db-path:.*|db-path: \"$HOME\/.sui\/db\"| ; s|genesis-file-location:.*|genesis-file-location: \"$HOME\/.sui\/genesis.blob\"| ; s|127.0.0.1|0.0.0.0|" $HOME/.sui/fullnode.yaml
```

## 7. Build SUI binaries.
```
cargo build --release
mv ~/sui/target/release/sui-node /usr/local/bin/
mv ~/sui/target/release/sui /usr/local/bin/
sui-node -V && sui -V
```


## 8. Create Service file for SUI Node.
```
echo "[Unit]
Description=Sui Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=/usr/local/bin/sui-node --config-path $HOME/.sui/fullnode.yaml
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/suid.service

mv $HOME/suid.service /etc/systemd/system/

sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```

## 9. Start SUI Full Node in Service.
```
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable suid
sudo systemctl restart suid
```

## 10. Check Logs
```
journalctl -u suid -f
```


how to check node status?
<br>
here: https://www.scale3labs.com/check/sui
<br>
or
<br>
```
curl -s -X POST http://127.0.0.1:9000 -H 'Content-Type: application/json'   --data-raw '{ "jsonrpc":"2.0", "method":"sui_getTotalTransactionNumber","id":1}' | jq 
```
