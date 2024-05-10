# MWC-Pool: Intro
- Forked from https://github.com/mining-pool/open-grin-pool
- Built for Ubuntu 22.04

### features
- relay the miner conn to the MWC node, totally native experience
- expose the TUI miner detail to Http API
- backup the share(submit) histories per specific time interval
- record the miner's pay method (manually sent coin by pool maintainer)
- pool status


### Install MWC node
```bash
sudo apt-get -y update && sudo apt-get -y upgrade
sudo apt-get -y install curl
snap install rustup --classic
rustup update
rustup default stable
sudo apt-get -y install build-essential cmake git libgit2-dev clang libncurses-dev libncurses5-dev libncursesw5-dev zlib1g-dev pkg-config libssl-dev llvm zlib1g-dev linux-headers-generic
cd /
mkdir MWC-Pool
cd /MWC-Pool
git clone https://github.com/mwcproject/mwc-node.git
cd mwc-node
cargo build --release
cp target/release/mwc ./mwc
nohup /MWC-Pool/mwc-node/mwc server run > /dev/null 2>&1 &
sleep 10
killall mwc
```


### Install MWC wallet
```bash
cd /MWC-Pool
git clone https://github.com/mwcproject/mwc-wallet.git
cd mwc-wallet
cargo build --release
cp target/release/mwc-wallet ./mwc-wallet
```


### Configure MWC node
```bash
### need to change "enable_stratum_server = false" to enable_stratum_server = true"
### need to uncomment the following:
# block_accepted_url = "http://127.0.0.1:8080/acceptedblock"
# tx_received_url = "http://127.0.0.1:8080/tx"
# header_received_url = "http://127.0.0.1:8080/header"
# block_received_url = "http://127.0.0.1:8080/block"
nano ~/.mwc/main/mwc-server.toml
```


### Configure MWC wallet
```bash
nano ~/.mwc/main/mwc-wallet.toml
```


### Run MWC node
```bash
nohup /MWC-Pool/mwc-node/mwc server run > /dev/null 2>&1 &
```


### Initialize MWC wallet
```bash
### creates a "password" file. Enter whatever password you want to use for the wallet moving forward. This file will be passed to the wallet when launched at startup.
nano /MWC-Pool/mwc-wallet/password.txt
cat /MWC-Pool/mwc-wallet/password.txt | /MWC-Pool/mwc-wallet/mwc-wallet init
```

### Run MWC wallet in listen mode
```bash
cat /MWC-Pool/mwc-wallet/password.txt | nohup /MWC-Pool/mwc-wallet/mwc-wallet listen > /dev/null 2>&1 &
```



# Install the Pool Environment

### install dependencies
```bash
sudo apt-get -y install redis-server apache2 nodejs npm golang-go
```

### ready (if using non-standard ports, be sure to allow them through the firewall via 'sudo ufw allow <port #>')
#### there may be several other ports that need to be opened up, which are not included here. Will need to parse through the config / toml files to discover them ###
```bash
echo >> /etc/apache2/apache2.conf
echo 'ServerName localhost' >> /etc/apache2/apache2.conf
sudo ufw allow 'Apache'
sudo ufw allow http
sudo ufw allow https
sudo ufw allow ssh
sudo ufw allow 4444
sudo ufw allow 3333 
sudo ufw allow 3413
sudo ufw allow 3415
sudo ufw allow 3416
sudo ufw enable
cd /MWC-Pool
git clone https://github.com/Mute-Whistle/MWC-pool.git pool
cd pool
```

### configure
#### Need to figure out how to pass the api keys from the node and wallet config files to the pool's config.json, for the auth_pass values under the "node" and "wallet" sections. Ideally we can do this using their local file paths, rather than plain text. They are located at:
### ~/.mwc/main/.api_secret
### ~/.mwc/main/.foreign_api_secret
### ~/.mwc/main/.owner_api_secret
```bash
nano config.json
```

### build
```bash
go build .
rm /var/www/html/index.html
cp -R web/* /var/www/html/
```

# start
```bash
nohup /MWC-Pool/pool/MWC-Pool > /dev/null 2>&1 &
sudo systemctl restart apache2
systemctl daemon-reload
```

WebAPI:

- `/pool` basic pool status
        example: curl 127.0.0.1:3333/pool
  
- `/revenue` the revenue **last day**, which the pool maintainer has to sent **today**
        example: curl 127.0.0.1:3333/revenue
  
- `/shares` all the miners' shares **today**
        example: curl 127.0.0.1:3333/shares

  - `/blocks` all the miners' blocks **today**
        example: curl 127.0.0.1:3333/blocks
  
- `/miner/{miner_login}` GET is the miner status
POST upload the payment method. e.g. ` curl 127.0.0.1:3333/miner/MinerWorkerName` will get the json of "MinerWorkerName"'s status. 
        example: curl  -X POST -d "{'pass': 'passwordOfMinerWorkerName', 'pm': 'http://<IP>:<PORT>'}" 127.0.0.1:3333/miner/MinerWorkerName`

The maintainer can manually use this command to send the coin `/MWC-Pool/mwc-wallet/mwc-wallet send -d http://<IP>:<PORT>`. Note, ensure the receiver online before your sending.

WebPage:
- replace the API address to your API in the web/config.js
- move all files in web to your Nginx/Apache folder
- accessible via web browser at http://localhost/

### Config

#### For server

You can keep all default except `auth_pass`. The password can be found in the `.api_secret` file. 
    
knowledge about this, check [here](https://github.com/mimblewimble/grin/blob/master/doc/api/api.md)

The `diff` should be the same difficulty to what you configured in server's (not pool's) `.toml` config file.


#### For miner

In miner's config(a `.toml` config file), these 2 params are **required**

```toml
# login for the stratum server (if required)
stratum_server_login = "loginName"

# password for the stratum server (if required)
stratum_server_password = "loginPass"
```

The `agent` is optional. Pool treat it as the miner's rig name.

**or the miner will not be able to connect to the server and the sol will not be relayed to the node!**

### TODO - Needs further development
- Web UI
- More accurate hash rate
- Administration based on web
- multi Backend (failover)
- multi Port (with different difficulties)


