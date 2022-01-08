* wasmd: v0.21.0
* testnet: Sandynet-1
* CosmJS: v0.26.0

testnet source

```bash
source <(curl -sSL https://raw.githubusercontent.com/CosmWasm/testnets/master/sandynet-1/defaults.env)
```
Setting up env

```bash
export NODE="--node $RPC"
export TXFLAG="${NODE} --chain-id ${CHAIN_ID} --gas-prices 0.025ujunox --gas auto --gas-adjustment 1.3"
```

launch CLI

```bash
npx @cosmjs/cli@^0.26 --init https://raw.githubusercontent.com/InterWasm/cw-plus-helpers/main/base.ts --init https://raw.githubusercontent.com/InterWasm/cw-plus-helpers/main/cw20-base.ts
```
create a wallet

```javascript
const [addr, client] = await useOptions(uniOptions).setup(YOUR_PASSWORD_HERE);
//> Getting ujunox from faucet
```

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

Check balance

```javascript
client.getBalance(addr,'ujunox')
//> { denom: 'ujunox', amount: '2000000000' }
```