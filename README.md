# Node and Validator Setup Docs

This is Andromachain's documentation to set up a node and validator.

## Prerequisites

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install make build-essential git jq chrony -y
sudo apt install gcc -y
```

## Increase open files limit

```bash
sudo su -c "echo 'fs.file-max = 65536' >> /etc/sysctl.conf"
sudo sysctl -p
```

## Install Go

```bash
curl https://dl.google.com/go/go1.18.3.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf
```

## Update environment variables

```bash
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export GOBIN=$HOME/go/bin
export PATH=$PATH:/usr/local/go/bin:$GOBIN
EOF
```

```bash
source $HOME/.profile
```

## Download Ignite

```bash
curl https://get.ignite.com/cli! | bash
```

Or

```bash
https://docs.ignite.com/guide/install
```

## Clone source repository

```bash
git clone <andromaverse repo link>
cd <andromaverse binary>
git checkout <andromaverse binary latest version>
```

## Build the chain

```bash
ignite chain build
```

Initialize the chain

```bash
andromaversed init <MONIKER> --chain-id <chain id>
```

## Set up persistent peers

```bash
cd $HOME/.andromaversed/config/
nano config.toml
```

Add peers

```
persistent_peers = <validator-generated persistent peer>
```

## Copy genesis file

```bash
git clone <repo link>
cd testnets/<chain id>
cp genesis.json $HOME/. andromaversed/config
```

## Starting the network

Now start the network with a terminal command to ensure it runs

```bash
andromaversed start
```

## Create andromaversed.service

You should create a service to ensure the node can run in the background. First, exit from the terminal command "andromaversed start". Ctrl-C will do.

Create andromaversed.service file with the following. Make sure to replace USER and HOME placeholders

```
[Unit]
Description=Andromaversed Node
After=network-online.target

[Service]
User=<USER>
ExecStart=<HOME>/go/bin/andromaversed start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

## Move file to systemd folder and enable the service

```bash
sudo mv andromaversed.service /etc/systemd/system/andromaversed.service
sudo systemctl enable andromaversed.service && sudo systemctl start andromaversed.service
```

## Check node info

```bash
curl -s localhost:26657/status | jq .result.sync_info.catching_up
#true output is syncing - false is synced
curl -s localhost:26657/status | jq .result.sync_info.latest_block_height
#this output is your last block synced
curl -s "http://:26657/status?" | jq .result.sync_info.latest_block_height
#this output the public node last block synced
```

## Create or recover keys

```bash
# Create new key
andromaversed keys add <KEY_NAME> --keyring-backend os
```

```bash
# Recover key
andromaversed keys add <MONIKER> --keyring-backend os —-recover
```

## Get test tokens

TBD

## Create Validator

```bash
andromaversed tx staking create-validator \
 --amount=100000000uandr \
 --pubkey=$(andromaversed tendermint show-validator) \
 --moniker=<your-moniker> \
 —chain-id=<chain id> \
 --commission-rate="0.05" \
 --commission-max-rate="0.10" \
 --commission-max-change-rate="0.05" \
 --min-self-delegation="1" \
 --gas="700000" \
 --from=(keyofyourvalidator)
```
