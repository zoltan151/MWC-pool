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
cd MWC-Pool
git clone https://github.com/mwcproject/mwc-node.git
cd mwc-node
cargo build --release
cp target/release/mwc ./mwc
```


### Install MWC wallet
```bash
cd /MWC-Pool
git clone https://github.com/mwcproject/mwc-wallet.git
cd mwc-wallet
cargo build --release
cp target/release/mwc-wallet ./mwc-wallet
/MWC-Pool/mwc-wallet/mwc-wallet init
### enter new password for the wallet ###
```


### Configure MWC node
```bash
nano ~/.mwc/main/mwc-server.toml
### need to change "enable_stratum_server = false" to enable_stratum_server = true"
### need to uncomment the following:
# block_accepted_url = "http://127.0.0.1:8080/acceptedblock"
# tx_received_url = "http://127.0.0.1:8080/tx"
# header_received_url = "http://127.0.0.1:8080/header"
# block_received_url = "http://127.0.0.1:8080/block"
```


### Configure MWC wallet
```bash
nano ~/.mwc/main/mwc-wallet.toml
```


### Usage

```bash
### for below commands for mwc-node and mwc-wallet, need to find a way to run them in the background, rather than the foreground. That way we can continue with other scripts, as well as set the node to launch at startup without manual intervention or necessitating a detached screen session ###
# run server
nohup /MWC-Pool/mwc-node/mwc server run > /dev/null 2>&1 &
# run wallet
nohup /MWC-Pool/mwc-wallet/mwc-wallet listen > /dev/null 2>&1 &
### enter mwc-wallet password created during init stage


# install dependencies
sudo apt-get -y install redis-server apache2 nodejs npm

# ready (if using non-standard ports, be sure to allow them through the firewall via 'sudo ufw allow <port #>')
### there may be several other ports that need to be opened up, which are not included here. Will need to parse through the config / toml files to discover them ###
sudo ufw allow 'Apache'
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 4444
sudo ufw allow 3333
sudo ufw enable
cd /MWC-Pool
git clone https://github.com/zoltan151/MWC-pool.git pool
cd pool

# configure
### Need to figure out how to pass the api keys from the node and wallet config files to the pool's config.json, for the auth_pass values under the "node" and "wallet" sections. Ideally we can do this using their local file paths, rather than plain text. They are located at:
### ~/.mwc/main/.api_secret
### ~/.mwc/main/.foreign_api_secret
### ~/.mwc/main/.owner_api_secret
nano config.json

# build
go build .
rm /var/www/html/index.html
cp -R web/* /var/www/html/

# start
### for below command for MWC-Pool, need to find a way to run them in the background, rather than the foreground. That way we can continue with other scripts, as well as set the node to launch at startup without manual intervention or necessitating a detached screen session ###
/MWC-Pool/pool/MWC-Pool
sudo systemctl restart apache2

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

Yyou can keep all default except `auth_pass`. The password can be found in the `.api_secret` file. 
    
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

### TODO
- Web UI
- More accurate hash rate
- Administration based on web
- multi Backend (failover)
- multi Port (with different difficulties)


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fmaoxs2%2Fopen-grin-pool.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fmaoxs2%2Fopen-grin-pool?ref=badge_large)
