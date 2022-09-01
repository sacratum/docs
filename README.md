# docs
# START HERE

sudo systemctl stop evmosd

evmosd unsafe-reset-all

peers="d2f1e656e9cbd06460ca3ed1acf3af09c00884d4@62.171.191.122:26656,be7593d1d2cae15a574537f9107f525200824767@194.163.187.94:26656" 

sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.evmosd/config/config.toml

SNAP_RPC="http://62.171.191.122:26657" #S3_NEW
SNAP_RPC2="http://194.163.187.94:26657" #S4

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
# outputs smth like this:
# 180751 178751 3D59C8106377431B25FFF841730C933FD15B110CB165B16526A348431F38CFB9

# if output OK!! do next

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC2\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" ~/.evmosd/config/config.toml



# configure pruning == start (OPTIONAL) ==============

sed -i.bak -e "s/^pruning = \"default\"/pruning = \"custom\"/" ~/.evmosd/config/app.toml
sed -i.bak -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" ~/.evmosd/config/app.toml
sed -i.bak -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"0\"/" ~/.evmosd/config/app.toml
sed -i.bak -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" ~/.evmosd/config/app.toml

# configure pruning == end ===========================


# restart node
sudo systemctl restart evmosd

journalctl -u evmosd -f
# wait until snapshot discovered and applied (about 15 min)

# wait for sync
curl -s localhost:26657/status

# after "catching_up: false" check several times 

evmosd status --node http://arsiamons.rpc.evmos.org:26657 | jq .SyncInfo.latest_block_height && \
evmosd status | jq .SyncInfo.latest_block_height

# your height must match with trusted node


# check validator
evmosd query staking validator АДРЕСВАЛИДАТОРА

# if jailed
evmosd tx slashing unjail --from ИМЯКОШЕЛЬКА --gas=auto --fees=1000aphoton --chain-id=evmos_9000-1

# END
