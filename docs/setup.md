# Setup Instruction Manual

These instructions cover the setup required for the instructor running the Stratum V2 workshop.

## Overview
The workshop setup includes:
1. A Genesis node that is publicly accessible for participants to sync their Bitcoin node with.
2. A Signet block explorer to display participants' mined blocks.

## Prerequisites
1. Install Rust:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
2. Install [Docker](https://docs.docker.com/engine/install/).

## Software Compatibility
* `bitcoin-core`:
  * Source: [Sjors's `sv2-tp-0.1.3` tag](https://github.com/Sjors/bitcoin/tree/sv2-tp-0.1.3).
  * Release Binary: [Plebhash's fork of Sjors's sv2-tp-0.1.3 tag](https://github.com/plebhash/bitcoin/releases/tag/btc-prague).
* `cpuminer` `v2.5.1`: [GitHub release](https://github.com/pooler/cpuminer/releases/tag/v2.5.1).
* `stratum` - `workshop` branch: [GitHub repo](https://github.com/stratum-mining/stratum/tree/workshop).

## `bitcoin-core`

### Purpose
A Bitcoin node is needed for:
1. Syncing participants' Bitcoin nodes with a Genesis node.
2. Running a block explorer to view mined blocks.

### Install
There are two ways to install the required `bitcoin-core` fork:

1. Download and extract the binary from [Plebhash's fork](https://github.com/plebhash/bitcoin/releases/tag/btc-prague).
2. Clone and build from Sjors's source:

```sh
git clone https://github.com/Sjors/bitcoin.git
cd bitcoin
git fetch --all
git checkout sv2-tp-0.1.3
./autogen.sh
./configure --disable-tests --disable-bench --enable-wallet --with-gui=no
make  # or `make -j <num cores>`
```

> Note: For mac users, it is highly recommended to build from source.

### Config

#### Genesis Node
A Genesis node that is publicly accessible is needed for participants to sync their Bitcoin nodes. This can be set up by the instructor or use the existing SRI VM node.

Verify the node is running:

```sh
ps -ef | grep -i bitcoind
> sri 3935787 1 0 Jun29 ? 01:19:13 /home/sri/btc_prague_workshop/bitcoin/src/bitcoind -signet -datadir=/home/sri/btc_prague_workshop/bitcoin_data_dir/ -fallbackfee=0.01 -daemon -sv2 -sv2port=38442
```

Ensure the `bitcoin.conf` in the `datadir` contains:

```conf
[signet]
signetchallenge=51    # OP_TRUE
prune=0
txindex=1
server=1
rpcallowip=0.0.0.0/0
rpcbind=0.0.0.0
rpcuser=mempool
rpcpassword=mempool
rpcport=38332
```

#### Block Explorer Node
A `signet` block explorer is needed to display participants' mined blocks.

Ensure the `bitcoin.conf` in the `datadir` contains:

```
[signet]
signetchallenge=51      # OP_TRUE
prune=0
txindex=1
server=1
connect=75.119.150.111  # Genesis Node
rpcallowip=0.0.0.0/0
rpcbind=0.0.0.0
rpcuser=mempool
rpcpassword=mempool
```

Run the Bitcoin node:

```sh
bitcoind -datadir=$HOME/.bitcoin-sv2-workshop -signet -sv2
```

## `electrs`

### Install
Clone and configure:

```sh
git clone https://github.com/romanz/electrs
cd electrs
git checkout v0.10.5
cat << EOF > electrs.toml
network="signet"
auth="mempool:mempool"
EOF
```

### Run
Run the server:

```sh
cargo run -- --signet-magic=54d26fbd
```

## `mempool.space`

### Install
Clone the repository:

```sh
git clone https://github.com/mempool/mempool
cd mempool
git checkout v2.5.0
```

### Config
Update `mempool/docker/docker-compose.yaml`:

```yaml
api:
  environment:
    MEMPOOL_BACKEND: "electrum"
    ELECTRUM_HOST: "host.docker.internal"
    ELECTRUM_PORT: "60601"
    ELECTRUM_TLS_ENABLED: "false"
    CORE_RPC_HOST: "host.docker.internal"
    CORE_RPC_PORT: "38332"
    CORE_RPC_USERNAME: "mempool"
    CORE_RPC_PASSWORD: "mempool"
    DATABASE_ENABLED: "true"
```

### Run
```sh
docker-compose up
```