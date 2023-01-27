**1. Stop and delete container with volumes.**
```
cd $HOME/sui
sudo docker compose down --volumes
```

**2. Download genesis state file.**
```
wget -O $HOME/sui/genesis.blob  https://github.com/MystenLabs/sui-genesis/raw/main/testnet/genesis.blob
```

**3. Update Docker image tag:**
```
IMAGE="mysten/sui-node:2698314d139a3018c2333ddaa670a7cb70beceee"
sed -i.bak "s|image:.*|image: $IMAGE|" $HOME/sui/docker-compose.yaml
```

**4. Start SUI Full Node in the Docker container.**
```
docker-compose up -d
```

**5. Check logs from SUI Node container:**
```
docker ps -a
```
```
docker logs -f <container id>
```
