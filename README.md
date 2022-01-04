# CosmWasm Dev Academy

Learning CosmWasm with [CosmWasm Dev Academy](https://docs.cosmwasm.com/fr/dev-academy/intro)

## Setting up the environment

There's a Gitpod solution to launch a virtual dev environment but I don't get it working, and I don't want to waste time to make it working.

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
