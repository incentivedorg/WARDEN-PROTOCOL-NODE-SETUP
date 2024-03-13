Warden-protocol
Update system
```
sudo apt update
sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y
```
Install Go
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
Installation & Configuration
```
Build the wardend binary and initalize the chain home folder:
```
git clone --depth 1 --branch v0.1.0 https://github.com/warden-protocol/wardenprotocol/
cd  wardenprotocol/warden/cmd/wardend
go build
sudo mv wardend /usr/local/bin/
wardend init <custom_moniker>
```
Prepare the genesis file:
```
cd $HOME/.warden/config
rm genesis.json
wget https://raw.githubusercontent.com/warden-protocol/networks/main/testnet-alfama/genesis.json

# Set minimum gas price & peers
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uward"/' app.toml
sed -i 's/persistent_peers = ""/persistent_peers = "6a8de92a3bb422c10f764fe8b0ab32e1e334d0bd@sentry-1.alfama.wardenprotocol.org:26656,7560460b016ee0867cae5642adace5d011c6c0ae@sentry-2.alfama.wardenprotocol.org:26656,24ad598e2f3fc82630554d98418d26cc3edf28b9@sentry-3.alfama.wardenprotocol.org:26656"/' config.toml
```
(Recommended) Setup state sync
```
export SNAP_RPC_SERVERS="https://rpc.sentry-1.alfama.wardenprotocol.org:443,https://rpc.sentry-2.alfama.wardenprotocol.org:443,https://rpc.sentry-3.alfama.wardenprotocol.org:443"
export LATEST_HEIGHT=$(curl -s "https://rpc.alfama.wardenprotocol.org/block" | jq -r .result.block.header.height)
export BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000))
export TRUST_HASH=$(curl -s "https://rpc.alfama.wardenprotocol.org/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
```
Check that all variables have been set correctly:
```
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

# output should be similar to:
# 70694 68694 6AF4938885598EA10C0BD493D267EF363B067101B6F81D1210B27EBE0B32FA2A
```
Now you can add the state sync configuration to your config.toml:
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC_SERVERS\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.warden/config/config.toml
```
Start the node You can now start the node using the following command:
```
wardend start
```
Create a validator
If you want to create a validator in the testnet, follow the instructions in the Creating a validator section.

Create Wallet
```
wardend keys add wallet
```
faucet token
```
curl --json '{"address": "<your-address>"}' https://faucet.alfama.wardenprotocol.org
```
Check Banlance
```
wardend query bank balances wallet
```
Create a new validator
Once the node is synced and you have the required WARD, you can become a validator.

To create a validator and initialize it with a self-delegation, you need to create a validator.json file and submit a create-validator transaction.

Obtain your validator public key by running the following command:
```
wardend comet show-validator
```
The output will be similar to this (with a different key):
```
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}
```
Create validator.json file
```
nano validator.json
```
The validator.json file has the following format: Change your personal information accordingly
```
{    
"pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
"amount": "1000000uward,
"moniker": "your-node-moniker",
"identity": "eqlab testnet validator",
"website": "optional website for your validator",
"security": "optional security contact for your validator",
"details": "optional details for your validator",
"commission-rate": "0.1",
"commission-max-rate": "0.2",
"commission-max-change-rate": "0.01",
"min-self-delegation": "1"
}
```
Finally, we're ready to submit the transaction to create the validator:
```
wardend tx staking create-validator validator.json \
--from=<key-name> \
--chain-id=alfama \
--fees=500uward
```
Explorer
```
https://warden-explorer.paranorm.pro/warden/staking
```
Delegate Token to your own validator
```
    wardend tx staking delegate $(wardend keys show wallet --bech val -a)  1000000uward \
    --from=wallet \
    --chain-id=alfama \
    --fees=500uward
```
Withdraw rewards and commission from your validator
```
    wardend tx distribution withdraw-rewards $(wardend keys show wallet --bech val -a) \
    --from wallet \
    --commission \
    --chain-id=alfama \
    --fees=500uward
```
Unjail validator
```
    wardend tx slashing unjail \
    --from wallet \
    --commission \
    --chain-id=alfama \
    --fees=500uward
```
Services Management
```
    # Reload Service
    sudo systemctl daemon-reload
    # Enable Service
    sudo systemctl enable wardend
    # Disable Service
    sudo systemctl disable wardend
    # Start Service
    sudo systemctl start wardend
    # Stop Service
    sudo systemctl stop wardend
    # Restart Service
    sudo systemctl restart wardend
    # Check Service Status
    sudo systemctl status wardend
    # Check Service Logs
    sudo journalctl -u wardend -f --no-hostname -o cat
```
Backup Validator
```
     cat $HOME/.warden/config/priv_validator_key.json
```
Remove node
```
    sudo systemctl stop wardend && sudo systemctl disable wardend && sudo rm /etc/systemd/system/wardend.service && sudo systemctl daemon-reload && rm -rf $HOME/.warden && $HOME/wardenprotocol
```
DONE
