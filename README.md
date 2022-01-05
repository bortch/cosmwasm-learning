# CosmWasm Dev Academy

Learning CosmWasm with [CosmWasm Dev Academy](https://docs.cosmwasm.com/fr/dev-academy/intro)

## Setting up the environment

Requirement:

* install latest `Rust`: https://rustup.rs/
* run:
  
  ```bash
  rustup target add wasm32-unknown-unknown
  ```

* install Go: https://go.dev/doc/install
* don't forget to check PATH (in `.profile` or `.bashrc`)
  
  ```bash
  # go
  export PATH="$PATH:/usr/local/go/bin"
  export GOPATH="$HOME/go"
  export PATH="$PATH:$GOPATH/bin"
  ```

* clone and install `wasmd`

  ```bash
  git clone https://github.com/CosmWasm/wasmd.git
  cd wasmd
  # replace the vX.XX.X with the most stable version on https://github.com/CosmWasm/wasmd/releases
  git checkout vX.XX.X
  make install
  # verify the installation
  wasmd version
  ```

* if not already there, install
  
  ```bash
  sudo apt-get install jq curl
  ```

## Configuration of wasmd & Wallet

`wasmd` is a fork of Gaia Daemon `gaiad` which is the Cosmos Hub (ATOM token) based on Cosmos SDK.
[more info on Cosmos Hub](https://hub.cosmos.network/main/hub-overview/overview.html)

```bash
# more details on available action
wasmd --help
```

Get config file from CosmWasm public network `pebblenet` as source

```bash
source <(curl -sSL https://raw.githubusercontent.com/CosmWasm/testnets/master/pebblenet-1/defaults.env)
```

### Add a test Wallet

```bash
wasmd keys add wallet
>
- name: wallet
  type: local
  address: wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d
  pubkey: wasmpub1addwnpepqdp65uvmmgx4wlqrcq2375pavtxs6299hutrn8el0mut9qft4wcy5e846ad
  mnemonic: "drop excuse judge critic struggle indicate report they excess corn maximum diary eye couch term nothing frost infant engine hover silk scale violin offer"
  threshold: 0
  pubkeys: []

wasmd keys add wallet2
>
- name: wallet2
  type: local
  address: wasm1gykjd96p5ednmv4a6vuaqjxncc9h54elu3xvuw
  pubkey: wasmpub1addwnpepqgnym6s2tcjrr9hl2a23gz7cuxgazep5auwh2k9nay9zn5hczqnz75wkg3t
  mnemonic: "success fame venture involve impulse view pitch soul glass excess city large fashion legend there royal citizen basket family fantasy arrive hidden butter make"
  threshold: 0
  pubkeys: []

```

### Get tokens from Faucet

```bash
JSON=$(jq -n --arg addr $(wasmd keys show -a wallet) '{"denom":"upebble","address":$addr}') && curl -X POST --header "Content-Type: application/json" --data "$JSON" https://faucet.pebblenet.cosmwasm.com/credit
```

### Check account balance

first you need to set the following variable, otherwise you will have to define type in node, chain id and gas-prices details with every command you execute

```bash
export NODE="--node $RPC"
export TXFLAG="${NODE} --chain-id ${CHAIN_ID} --gas-prices 0.025upebble --gas auto --gas-adjustment 1.3"
```

To check the `wallet` account balance:

```bash
wasmd query bank balances $(wasmd keys show -a wallet) $NODE

> balances:
- amount: "2000000000"
  denom: upebble
  pagination: {}

```

## Smart Contract

### Sources

pre-compiled wasm contract: https://github.com/CosmWasm/cw-plus/releases

### Deploy

```bash
curl -LJO https://github.com/CosmWasm/cw-plus/releases/download/v0.8.0/cw20_base.wasm
RES=$(wasmd tx wasm store cw20_base.wasm --from wallet $TXFLAG -y)

> gas estimate: 1889175

echo $RES

> {"height":"2153100","txhash":"4266EF52A86C22EF0D0AE43AFB0A460723BAAD5C4AC057023B030ACD392102B7",  "data":"0A110A0A73746F72652D636F6465120308F702","raw_log":"[{\"events\":[{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"store-code\"},{\"key\":\"module\",\"value\":\"wasm\"},{\"key\":\"sender\",\"value\":\"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d\"}]},{\"type\":\"store_code\",\"attributes\":[{\"key\":\"code_id\",\"value\":\"375\"}]}]}]","logs":[{"events":[{"type":"message","attributes":[{"key":"action","value":"store-code"},{"key":"module","value":"wasm"},{"key":"sender","value":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d"}]},{"type":"store_code","attributes":[{"key":"code_id","value":"375"}]}]}],"gas_wanted":"1889214","gas_used":"1465933"}

# get code id of the stored contract
CODE_ID=$(echo $RES | jq -r '.logs[0].events[1].attributes[0].value')

# print code id
echo $CODE_ID

> 375

# no contracts yet, this should return an empty list
wasmd query wasm list-contract-by-code $CODE_ID $NODE --output json

> {"pagination":{}}
# deploying code doesn't means instanciate contract

```

Smart contract code is just a blueprint of a smart contract.
You need to instantiate a smart contract based on the deployed smart contract code.

### Smart Contract Instantiation

```json
{
  "name": "Golden Stars",
  "symbol": "STAR",
  "decimals": "2",
  "initial_balances": [
    {"address": "wasm1ez03me7uljk7qerswdp935vlaa4dlu48mys3mq", "amount": "10000"},
    {"address": "wasm1tx7ga0lsnumd5hfsh2py0404sztnshwqaqjwy8", "amount": "10000"},
    {"address": "here-will-load-our-wallet-address", "amount": "10000"}
  ],
  "mint": {
    "minter": "here-will-load-our-wallet-address"
  }
}
```

```bash
INIT=$(jq -n --arg wallet $(wasmd keys show -a wallet) '{"name":"Golden Stars","symbol":"STAR","decimals":2,"initial_balances":[{"address":"wasm1n8aqd9jq9glhj87cn0nkmd5mslz3df8zm86hrh","amount":"10000"},{"address":"wasm13y4tpsgxza44yq76qvj69sakq4jmeyqudwgwpk","amount":"10000"},{"address":$wallet,"amount":"10000"}],"mint":{"minter":$wallet}}')
wasmd tx wasm instantiate $CODE_ID "$INIT" --from wallet $TXFLAG --label "first cw20"

> gas estimate: 180677
{"body":{"messages":[{"@type":"/cosmwasm.wasm.v1.MsgInstantiateContract","sender":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d","admin":"","code_id":"375","label":"first cw20","msg":{"name":"Golden Stars","symbol":"STAR","decimals":2,"initial_balances":[{"address":"wasm1n8aqd9jq9glhj87cn0nkmd5mslz3df8zm86hrh","amount":"10000"},{"address":"wasm13y4tpsgxza44yq76qvj69sakq4jmeyqudwgwpk","amount":"10000"},{"address":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d","amount":"10000"}],"mint":{"minter":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d"}},"funds":[]}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[{"denom":"upebble","amount":"4517"}],"gas_limit":"180677","payer":"","granter":""}},"signatures":[]}

> confirm transaction before signing and broadcasting [y/N]: y
{"height":"2153182","txhash":"9B0F4D2DA0293FF4C61FB8A73A66BEEA39A99A46E5FFE1666FC6F984277E3250","data":"0A3C0A0B696E7374616E7469617465122D0A2B7761736D316178716A6E7479776672377272793434336475757077767A6B663268337074656A3861687268","raw_log":"[{\"events\":[{\"type\":\"instantiate\",\"attributes\":[{\"key\":\"_contract_address\",\"value\":\"wasm1axqjntywfr7rry443duupwvzkf2h3ptej8ahrh\"},{\"key\":\"code_id\",\"value\":\"375\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"instantiate\"},{\"key\":\"module\",\"value\":\"wasm\"},{\"key\":\"sender\",\"value\":\"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d\"}]}]}]","logs":[{"events":[{"type":"instantiate","attributes":[{"key":"_contract_address","value":"wasm1axqjntywfr7rry443duupwvzkf2h3ptej8ahrh"},{"key":"code_id","value":"375"}]},{"type":"message","attributes":[{"key":"action","value":"instantiate"},{"key":"module","value":"wasm"},{"key":"sender","value":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d"}]}]}],"gas_wanted":"180677","gas_used":"148754"}
```

Success JSON response

```json
{
  "height": "2153182",
  "txhash": "9B0F4D2DA0293FF4C61FB8A73A66BEEA39A99A46E5FFE1666FC6F984277E3250",
  "data": "0A3C0A0B696E7374616E7469617465122D0A2B7761736D316178716A6E7479776672377272793434336475757077767A6B663268337074656A3861687268",
  "raw_log": "[{\"events\":[{\"type\":\"instantiate\",\"attributes\":[{\"key\":\"_contract_address\",\"value\":\"wasm1axqjntywfr7rry443duupwvzkf2h3ptej8ahrh\"},{\"key\":\"code_id\",\"value\":\"375\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"instantiate\"},{\"key\":\"module\",\"value\":\"wasm\"},{\"key\":\"sender\",\"value\":\"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d\"}]}]}]",
  "logs": [{
    "events": [{
      "type": "instantiate",
      "attributes": [{
        "key": "_contract_address",
        "value": "wasm1axqjntywfr7rry443duupwvzkf2h3ptej8ahrh"
      }, {
        "key": "code_id",
        "value": "375"
      }]
    }, {
      "type": "message",
      "attributes": [{
        "key": "action",
        "value": "instantiate"
      }, {
        "key": "module",
        "value": "wasm"
      }, {
        "key": "sender",
        "value": "wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d"
      }]
    }]
  }],
  "gas_wanted": "180677",
  "gas_used": "148754"
}
```

Now we can find the contract:

```bash
wasmd query wasm list-contract-by-code $CODE_ID $NODE --output json
> {"contracts":["wasm1axqjntywfr7rry443duupwvzkf2h3ptej8ahrh"],"pagination":{}}
```

## Using CosmJS - REPL client

Retry from start using CosmJS CLI

### install & launch CosmJS

NodeJS required

```bash
npx @cosmjs/cli@^0.26 --init https://raw.githubusercontent.com/InterWasm/cw-plus-helpers/main/base.ts --init https://raw.githubusercontent.com/InterWasm/cw-plus-helpers/main/cw20-base.ts
```

### Create a wallet


```javascript
const [addr, client] = await useOptions(pebblenetOptions).setup(YOUR_PASSWORD_HERE);
> Getting upebble from faucet
```

#### Address info

```javascript
client.getAccount(addr);
```

it will show something like

```javascript
{
  address: 'wasm1euf9qpj7nvdu9w5swu0mvhwtgcnek528trs2lk',
  pubkey: null,
  accountNumber: 994,
  sequence: 0
}
```

#### Mnemonic

```javascript
//to get the Mnemonic
useOptions(pebblenetOptions).recoverMnemonic(YOUR_PASSWORD_HERE)
> 'excess amount hundred goose hope veteran crowd captain fossil merry cross quiz'
```

#### Balance

```javascript
client.getBalance(addr,'upebble')
> { denom: 'upebble', amount: '2000000000' }
```