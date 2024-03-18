# chain

## Before start

Copy all files from `build/credentials` with `example.*` prefix to same files without `example` prefix and correct values

**chain** is a blockchain built using Cosmos SDK and Tendermint and created with [Starport](https://github.com/tendermint/starport).

## Get started

```
starport serve
```

`serve` command installs dependencies, builds, initializes and starts your blockchain in development.

Also, you need to set up system mnemonic for runner here `build/credentials/VC.core.system.account.json`

## Configure

Your blockchain in development can be configured with `config.yml`. To learn more, see the [Starport docs](https://docs.starport.network).

## Launch

To launch your blockchain live on multiple nodes, use `starport network` commands. Learn more about [Starport Network](https://github.com/tendermint/spn).

## Init blockchain

To create 6 nodes (4 of them - validators) - `make init`

## Compose

To create and run 6 nodes (4 of them - validators) in docker-compose use `make compose-init`

## Add validator node after running blockchain
after blockchain start if we need add one more validator node

`make add-validator` (only after `make init`)

make sure, you already have a correct configuration for nodes

then, after blockchain start, we need get account address from
`VC_core_system_account_validator-node.json` file,
for example it is - `cosmos1dw62e3nryd3x7qmqftnw7gxmnmd4zv49svnjy5`
we need to send `stake` coins to this account `stake` coins

for example, do this from `cosmos-0` node
```shell
chaind tx bank send cosmos-0 cosmos1rvw0nycd46jtwwjx5d2g2jaratvrg3ssszvwcv 10000000stake --chain-id chain --home build/volumes/vc-core-v3d/cosmos-0
```
output example:
```shell
chaind tx bank send cosmos-0 vcg1zwazfnvg2mzjghczfhws3anzptas5f3wnqjr0t 1000stake --chain-id chain --home . --keyring-backend test
{"body":{"messages":[{"@type":"/cosmos.bank.v1beta1.MsgSend","from_address":"cosmos144qcjl65gmh2vdk82d8qww7mws2fkstgmxczgq","to_address":"cosmos1dw62e3nryd3x7qmqftnw7gxmnmd4zv49svnjy5","amount":[{"denom":"stake","amount":"10000000"}]}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[],"gas_limit":"200000","payer":"","granter":""}},"signatures":[]}

confirm transaction before signing and broadcasting [y/N]: y
{"height":"1211","txhash":"46F2558451662BC1CA7812C37AB9B0A7B839258F815CCA92ECF6B06A5EC4BCA4","codespace":"","code":0,"data":"0A060A0473656E64","raw_log":"[{\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"send\"},{\"key\":\"sender\",\"value\":\"cosmos144qcjl65gmh2vdk82d8qww7mws2fkstgmxczgq\"},{\"key\":\"module\",\"value\":\"bank\"}]},{\"type\":\"transfer\",\"attributes\":[{\"key\":\"recipient\",\"value\":\"cosmos1dw62e3nryd3x7qmqftnw7gxmnmd4zv49svnjy5\"},{\"key\":\"sender\",\"value\":\"cosmos144qcjl65gmh2vdk82d8qww7mws2fkstgmxczgq\"},{\"key\":\"amount\",\"value\":\"10000000stake\"}]}]}]","logs":[{"msg_index":0,"log":"","events":[{"type":"message","attributes":[{"key":"action","value":"send"},{"key":"sender","value":"cosmos144qcjl65gmh2vdk82d8qww7mws2fkstgmxczgq"},{"key":"module","value":"bank"}]},{"type":"transfer","attributes":[{"key":"recipient","value":"cosmos1dw62e3nryd3x7qmqftnw7gxmnmd4zv49svnjy5"},{"key":"sender","value":"cosmos144qcjl65gmh2vdk82d8qww7mws2fkstgmxczgq"},{"key":"amount","value":"10000000stake"}]}]}],"info":"","gas_wanted":"200000","gas_used":"57686","tx":null,"timestamp":""}
```
after that, check balance of account
```shell
chaind query bank balances cosmos1rvw0nycd46jtwwjx5d2g2jaratvrg3ssszvwcv --chain-id chain --home build/volumes/vc-core-v3d/cosmos-0
```
output example:
```shell
chaind query bank balances cosmos1dw62e3nryd3x7qmqftnw7gxmnmd4zv49svnjy5
balances:
- amount: "10000000"
  denom: stake
pagination:
  next_key: null
  total: "0"
```
then we need to get tendermint key from generated validator node
```shell
chaind tendermint show-validator --home ./build/volumes/vc-core-v3d/validator-node
```
now we can run generated `validator-node`
```shell
chaind start --home ./build/volumes/vc-core-v3d/validator-node/
```
wait for sync (state_sync and fast_sync description below), then we need to sign and send tx `create-validator` to any running node, for example `cosmos-0`
```shell
chaind tx staking create-validator \
--amount=10000000stake \
--pubkey="cosmosvalconspub1zcjduepqha49raw700h0s6gsdjpe4p8gv9w8y0ddh5tu7ja9k9yky645cq8qrklhrk" \
--moniker=cosmos-4 \
--chain-id=chain \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--gas="auto" \
--gas-adjustment=1.15 \
--from=cosmos-4 --home ./build/volumes/vc-core-v3d/cosmos-4/
```
its all, you can verify adding validator by routes:
```http request
http://cosmos-0:1317/cosmos/base/tendermint/v1beta1/blocks/latest
```
```http request
http://cosmos-0:1317/cosmos/slashing/v1beta1/signing_infos
```
```http request
http://cosmos-0:1317/cosmos/staking/v1beta1/validators
```
when validator node comes to offline for long time, he can be jailed automatically,
for unjail you need to run on jailed node:
```shell
chaind tx slashing unjail --from testnode --chain-id chain
```

## Fast sync
enabled by default, for using it, you need to write your nodes into `persistant_peers`
example:
```shell
persistent_peers = "95f631b5e8967d5c8c55657caa5cbb184a34d2c7@cosmos-1:26656,efadbeae5174e2fe5a09adbda00c42deee188627@cosmos-2:26656,23e7829b3efe7bad804750e980d79b5f5fe8043e@cosmos-3:26656,6ff00476983206b5f92e7109b6e6dbe358b8b550@cosmos-4:26656,2d1ac88860740469ec871f5c4988f61d12640386@cosmos-5:26656,4788ddf2ac1d88c77dbd7238bd2ab651d432694e@cosmos-6:26656"

`config.yaml`
```shell
[statesync]
enable = true
...
rpc_servers = "tcp://cosmos-1:26657,tcp://cosmos-2:26657,tcp://cosmos-3:26657,tcp://cosmos-4:26657,tcp://cosmos-5:26657,tcp://cosmos-6:26657"
trust_height = 102
trust_hash = "1478C352DF218F388D1EDE4F7CB86236EAB2A67B3B5431ED300EAC673A5298B8"
```
`trust_height` and `trust_hash` we can get from (required `jq` or remove from cmd)
```shell
curl -s http://cosmos-0:26657/commit | jq "{height: .result.signed_header.header.height, hash: .result.signed_header.commit.block_id.hash}"
```

## Web Frontend

Starport has scaffolded a Vue.js-based web app in the `vue` directory. Run the following commands to install dependencies and start the app:

```shell
cd vue
npm install
npm run serve
```

The frontend app is built using the `@starport/vue` and `@starport/vuex` packages. For details, see the [monorepo for Starport front-end development](https://github.com/tendermint/vue).

## Learn more

- [Starport](https://github.com/tendermint/starport)
- [Starport Docs](https://docs.starport.network)
- [Cosmos SDK documentation](https://docs.cosmos.network)
- [Cosmos SDK Tutorials](https://tutorials.cosmos.network)
- [Discord](https://discord.gg/W8trcGV)
