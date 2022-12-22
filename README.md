<h1 align="center"> Gitopia-State-Sync </h1>
Gitopia'da bloklarınızın hızlıca eşleşmesini istiyorsanız uygulayabilirsiniz. Çok işe yarıyor.


- Node'umuzu durduruyoruz ve verileri sıfırlıyoruz.

```
sudo systemctl stop gitopiad
```

```
cp $HOME/.gitopia/data/priv_validator_state.json $HOME/.gitopia/priv_validator_state.json.backup
gitopiad tendermint unsafe-reset-all --home $HOME/.gitopia
```

- Durum eşitleme bilgilerini alıyoruz ve yapılandırma yapıyoruz.

```
STATE_SYNC_RPC=https://gitopia-testnet.rpc.kjnodes.com:443
STATE_SYNC_PEER=d5519e378247dfb61dfe90652d1fe3e2b3005a5b@gitopia-testnet.rpc.kjnodes.com:41656
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 2000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.gitopia/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
  $HOME/.gitopia/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
  $HOME/.gitopia/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
  $HOME/.gitopia/config/config.toml
sed -i.bak -e "s|^persistent_peers *=.*|persistent_peers = \"$STATE_SYNC_PEER\"|" \
  $HOME/.gitopia/config/config.toml
mv $HOME/.gitopia/priv_validator_state.json.backup $HOME/.gitopia/data/priv_validator_state.json
```

- Yeniden başlatıyoruz ve logları kontrol ediyoruz.

```
sudo systemctl start gitopiad && journalctl -u gitopiad -f --no-hostname -o cat
```

- Bu işlem sayesinde hem sunucumuzun hafızasında yer açılacak hem de bloklar daha hızlı eşleşecek.
