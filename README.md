# CosmWasm, wasmd, cosmJS

## Learning Sources

CosmWasm website has the [CosmWasm Dev Academy](https://docs.cosmwasm.com/fr/dev-academy/intro)

By the time I went through the tutorial, some updates had taken place. 

The tutorial was no longer fully functional as is.

As the whole thing changes quite rapidly, here are the versions I've used:

* rustc: 1.57.0
* go: 1.17.5 linux/amd64
* wasmd: v0.21.0
* testnet: Sandynet-1
* CosmJS: v0.26.0

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

## Configuration of wasmd

`wasmd` is a fork of Gaia Daemon `gaiad` which is the Cosmos Hub (ATOM token) based on Cosmos SDK.
[more info on Cosmos Hub](https://hub.cosmos.network/main/hub-overview/overview.html)

```bash
# more details on available action
wasmd --help
```

### Testnet

To choosing the last test network: https://github.com/CosmWasm/testnets

The last one for me was `Sandynet`

Requirement were:

* go version: 1.17.3
* wasmd version: v0.21.0
* CosmJS version: v0.26.0

Get config file from CosmWasm public test network:

```bash
source <(curl -sSL https://raw.githubusercontent.com/CosmWasm/testnets/master/sandynet-1/defaults.env)
```

### Set env variables

first you need to set the following variable, otherwise you will have to define type in node, chain id and gas-prices details with every command you execute

Make sure that the type of fees corresponds to the chosen testnet, for sandynet its defined as `ujunox`

```bash
export NODE="--node $RPC"
export TXFLAG="${NODE} --chain-id ${CHAIN_ID} --gas-prices 0.025ujunox --gas auto --gas-adjustment 1.3"
```

## Playing with wasmd

### Create a wallet

```bash
wasmd keys add wallet
#>
#- name: wallet
#  type: local
#  address: wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d
#  pubkey: wasmpub1addwnpepqdp65uvmmgx4wlqrcq2375pavtxs6299hutrn8el0mut9qft4wcy5e846ad
#  mnemonic: "drop excuse judge critic struggle indicate report they excess corn maximum diary eye couch term nothing frost infant engine hover silk scale violin offer"
#  threshold: 0
#  pubkeys: []

wasmd keys add wallet2
#>
#- name: wallet2
#  type: local
#  address: wasm1gykjd96p5ednmv4a6vuaqjxncc9h54elu3xvuw
#  pubkey: wasmpub1addwnpepqgnym6s2tcjrr9hl2a23gz7cuxgazep5auwh2k9nay9zn5hczqnz75wkg3t
#  mnemonic: "success fame venture involve impulse view pitch soul glass excess city large fashion legend there royal citizen basket family fantasy arrive hidden butter make"
#  threshold: 0
#  pubkeys: []

```

### Get tokens from Faucet

```bash
JSON=$(jq -n --arg addr $(wasmd keys show -a wallet) '{"denom":"ujunox","address":$addr}') && curl -X POST --header "Content-Type: application/json" --data "$JSON" https://faucet.sandynet.cosmwasm.com/credit
```

### Check account balance

To check the `wallet` account balance:

```bash
wasmd query bank balances $(wasmd keys show -a wallet) $NODE

> balances:
- amount: "2000000000"
  denom: ujunox
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
{"body":{"messages":[{"@type":"/cosmwasm.wasm.v1.MsgInstantiateContract","sender":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d","admin":"","code_id":"375","label":"first cw20","msg":{"name":"Golden Stars","symbol":"STAR","decimals":2,"initial_balances":[{"address":"wasm1n8aqd9jq9glhj87cn0nkmd5mslz3df8zm86hrh","amount":"10000"},{"address":"wasm13y4tpsgxza44yq76qvj69sakq4jmeyqudwgwpk","amount":"10000"},{"address":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d","amount":"10000"}],"mint":{"minter":"wasm1a555386zdv895w0jn7lfzfjjwkpgsnhjvq6j3d"}},"funds":[]}],"memo":"","timeout_height":"0","extension_options":[],"non_critical_extension_options":[]},"auth_info":{"signer_infos":[],"fee":{"amount":[{"denom":"ujunox","amount":"4517"}],"gas_limit":"180677","payer":"","granter":""}},"signatures":[]}

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

### Details

to get implementation details check docstring from :

* https://raw.githubusercontent.com/InterWasm/cw-plus-helpers/main/cw20-base.ts
* https://raw.githubusercontent.com/InterWasm/cw-plus-helpers/main/base.ts

They contain, among other things:

* methods names
* option name: `uniOptions` for sandynet-1 
* `feeToken`'s name: `'ujunox'` for sandynet-1

### Create a wallet

```javascript
let PASSWORD = "YOUR_PASSWORD_HERE";
const [addr, client] = await useOptions(uniOptions).setup(PASSWORD);
//> Getting ujunox from faucet
```

#### Address info

```javascript
client.getAccount(addr);
```

It will show something like:

```javascript
{
  address: 'juno16s9cq4g6zrzflmuchqn7rk5nddhad696fudtcj',
  pubkey: {
    type: 'tendermint/PubKeySecp256k1',
    value: 'Al2iFzOASuyHNqpVTe1HL3HZRjLXpQHZzfuU8rahy0kD'
  },
  accountNumber: 52949,
  sequence: 1
}
```

#### Mnemonic

```javascript
//to get the Mnemonic
useOptions(uniOptions).recoverMnemonic(PASSWORD)
//> 'web vague cattle sauce poet snow shadow cause once case gather foster'
```

#### Balance

```javascript
client.getBalance(addr,'ujunox')
//> { denom: 'ujunox', amount: '2000000000' }
```

### Deploy a Smart Contract

```javascript
// always start with setting client and address
let PASSWORD = "YOUR_PASSWORD_HERE";
const [addr, client] = await useOptions(uniOptions).setup(PASSWORD);

// deploy the smart contract
const cw20 = CW20(client, uniOptions);
// keep codeID
const codeId = await cw20.upload(addr,uniOptions);
console.log(`CodeId: ${codeId}`);
// CodeId: 307
```

The smart contract is created and uploaded, but isn't yet instantiated

### Set Smart Contract instance params

```javascript
// enable REPL editor mode to edit multiline code then execute
.editor

// define smart contract instance parameters
const initMsg = {
  name: "My Coin",
  symbol: "MYCOIN",
  decimals: 5, // divide amount by 10^{decimals}

  // list of all validator self-delegate addresses - 100 STARs each!
  initial_balances: [
    {address: addr, amount: "123400000"},
  ],
  mint: {
    minter: addr,
    cap:"999900000"
  },
};
// exit editor using `ctrl+D` and execute entered code
```

### Instantiate Smart Contract

```javascript
const mine = await cw20.instantiate(addr, codeId, initMsg, "MYCOIN", uniOptions);
console.log(`Contract: ${mine.contractAddress}`);
// Contract: juno1glnjv27xrjtshcs7legz7e50hjtqr6fpzdpgnmmqyzgzx4tgqc4qmddqap

console.log(await mine.balance(mine.contractAddress));
// 0

// now, check the configuration
mine.balance(addr);
//> '123400000'

mine.tokenInfo()
/*
{
  name: 'My Coin',
  symbol: 'MYCOIN',
  decimals: 5,
  total_supply: '123400000'
}
*/

mine.minter()
/*
{
  minter: 'juno16s9cq4g6zrzflmuchqn7rk5nddhad696fudtcj',
  cap: '999900000'
}
*/
```

### Playing with contract

```javascript
let PASSWORD = "YOUR_PASSWORD_HERE";
const [addr, client] = await useOptions(uniOptions).setup(PASSWORD);
const cw20 = CW20(client, uniOptions);

// if you forgot your address, but remember your label, you can find it again
// getContracts by codeID
const codeID = 307; // from smart contract deployment
const contracts = await client.getContracts(codeID);
contracts
//> [ 'juno1glnjv27xrjtshcs7legz7e50hjtqr6fpzdpgnmmqyzgzx4tgqc4qmddqap' ]

// if more than once, get address by filtering on contracts
// const contractAddress = contracts.filter(x => x.label === 'MYCOIN')[0].address;

// else
const contractAddress = contracts[0]
// otherwise, you can just cut and paste from before
// const contractAddress = "juno1glnjv27xrjtshcs7legz7e50hjtqr6fpzdpgnmmqyzgzx4tgqc4qmddqap"

// now, connect to that contract and make sure it is yours
const mine = cw20.use(contractAddress);
mine.tokenInfo()
/*
{
  name: 'My Coin',
  symbol: 'MYCOIN',
  decimals: 5,
  total_supply: '123400000'
}
*/
mine.minter()
/*
{
  minter: 'juno16s9cq4g6zrzflmuchqn7rk5nddhad696fudtcj',
  cap: '999900000'
}
*/
mine.balance(addr)
//> '123400000'
```

### Mine & transfer token

```javascript
// get 2 address
const someone = "wasm13nt9rxj7v2ly096hm8qsyfjzg5pr7vn56p3cay";
const other = "wasm1ve2n9dd4uy48hzjgx8wamkc7dp7sfdv82u585d";

// right now, only you have tokens
mine.balance(addr)
mine.balance(someone)
mine.balance(other)
// and watch the total
mine.tokenInfo()

// let's mint some tokens for someone
mine.mint(addr, someone, "999888000")
// Bonus, take the tx hash printed out and cut-paste that into https://bigdipper.wasmnet.cosmwasm.com
// eg 26D5514CF437EE584793768B56CB4E605F1F6E61FC0123030DC64E08E2EE97FA

// See balances updated
mine.balance(someone)
mine.balance(addr)
// and the supply goes up
mine.tokenInfo()

// Okay, now let's transfer some tokens... that is the more normal one, right?
mine.transfer(addr, other, "4567000");
// eg. 4A76EFFEB09C82D0FEB97C3B5A9D5BADB6E9BD71F4EF248A3EF8B232C2F7262A
mine.balance(other)
mine.balance(addr)
```
