# CosmWasm Dev Academy

Learning CosmWasm with [CosmWasm Dev Academy](https://docs.cosmwasm.com/fr/dev-academy/intro)

## Setting up the environment

There's a Gitpod solution to launch a virtual dev environment and avoid to mess with yours.

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/bortch/cosmwasm-learning)

Requirement:

* install last `Rust`: https://rustup.rs/
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

## Configuration of wasmd & Wallet

`wasmd` is a fork of Gaia Daemon `gaiad` which is the Cosmos Hub (ATOM token) based on Cosmos SDK.
[more info on Cosmos Hub](https://hub.cosmos.network/main/hub-overview/overview.html)

```bash
# more details on available action
wasmd --help
```

Récupérer le fichier contenant les configurations de CosmWasm du réseau de test public `pebblenet`

```bash
source <(curl -sSL https://raw.githubusercontent.com/CosmWasm/testnets/master/pebblenet-1/defaults.env)
```

