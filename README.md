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
export TXFLAG="${NODE} --chain-id ${CHAIN_ID} --gas-prices 0.025ubay --gas auto --gas-adjustment 1.3"
```

To check the `wallet` account balance:

```bash
wasmd query bank balances $(wasmd keys show -a wallet) $NODE

> balances:
- amount: "2000000000"
  denom: upebble
  pagination: {}

```
