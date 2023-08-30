# Polkadot Substrate Development Bootcamp Final Project

- [Polkadot Substrate Development Bootcamp Final Project](#polkadot-substrate-development-bootcamp-final-project)
  - [Install tolls](#install-tolls)
  - [Building a blockchain](#building-a-blockchain)
    - [Cloned Repositories](#cloned-repositories)
  - [Simulating a substrate network](#simulating-a-substrate-network)
  - [Adding trusted nodes to a network](#adding-trusted-nodes-to-a-network)
    - [Run Node01](#run-node01)
    - [Add Participants - Node02](#add-participants---node02)
  - [Smart contracts](#smart-contracts)
    - [Update Rust Toolchain](#update-rust-toolchain)
    - [Install cargo-contract CLI Tools](#install-cargo-contract-cli-tools)
    - [Install the Substrate Contracts Node](#install-the-substrate-contracts-node)
    - [Smart Contract](#smart-contract)
      - [Create a Smart Contract](#create-a-smart-contract)
      - [Test The Contract](#test-the-contract)
      - [Build The Contract](#build-the-contract)
    - [Start the Substrate Contracts Node](#start-the-substrate-contracts-node)
    - [Deploy the contract](#deploy-the-contract)
      - [Uploading the ink! Contract Code](#uploading-the-ink-contract-code)

## Install tolls

[Install Tools for Substrate Development](https://docs.substrate.io/install/)

## Building a blockchain

[Substrate Documentation](https://docs.substrate.io/tutorials/build-a-blockchain/build-local-blockchain/)

### Cloned Repositories

- [substrate-node-template](https://github.com/substrate-developer-hub/substrate-node-template)
- [substrate-front-end-template](https://github.com/substrate-developer-hub/substrate-front-end-template)

For building a blockchain, we will be using the substrate-node-template repository. This repository contains the code for a basic Substrate node.

```bash
git clone https://github.com/substrate-developer-hub/substrate-node-template
cd substrate-node-template
git switch -c branch-name
cargo build --release
./target/release/node-template --dev
```

The above commands will clone the substrate-node-template repository, create a new branch, build the node-template, and run the node-template in development mode.

For Simulating a substrate network, we will be using the substrate-front-end-template repository. This repository contains the code for a basic Substrate front-end.

```bash
git clone https://github.com/substrate-developer-hub/substrate-front-end-template
cd substrate-front-end-template
yarn install
yarn start
```

## Simulating a substrate network

[Substrate Documentation](https://docs.substrate.io/tutorials/build-a-blockchain/simulate-network/)

```bash
./target/release/node-template purge-chain --base-path /tmp/alice --chain local -y
Are you sure to remove "/tmp/alice/chains/local_testnet/db"? [y/N]: y
```

```bash
./target/release/node-template \
--base-path /tmp/alice \
--chain local \
--alice \
--port 30333 \
--rpc-port 9945 \
--node-key 0000000000000000000000000000000000000000000000000000000000000001 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator
```

```bash
./target/release/node-template purge-chain --base-path /tmp/bob --chain local -y
Are you sure to remove "/tmp/alice/chains/local_testnet/db"? [y/N]: y
```

```bash
./target/release/node-template \
--base-path /tmp/bob \
--chain local \
--bob \
--port 30334 \
--rpc-port 9946 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
```

## Adding trusted nodes to a network

[Substrate Documentation](https://docs.substrate.io/tutorials/build-a-blockchain/add-trusted-node/)

Before starting the node, we need a second sudo user. We will be using the second sudo user to start the node.

> [!IMPORTANT]
> Run these commands different terminals and different users

```bash
./target/release/node-template key generate --scheme Sr25519 --password-interactive
```

After command execution, you will enter a password don't forget it. This will generate a AURA key. Copy the generated Secret phrase and SS58 Address and save it somewhere safe We will use that phrase and address.

```bash
Secret phrase:  pig giraffe ceiling enter weird liar orange decline behind total despair fly
Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex):  0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Account ID:        0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Public key (SS58): 5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
SS58 Address:      5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
```

Paste the Secret Phrase in the following command and type the password when prompted. This will generate a GRANDPA key and copy the SS58 Address.

```bash
./target/release/node-template key inspect --password-interactive --scheme Ed25519 "your secret phrase"
```

```bash
Secret phrase:  pig giraffe ceiling enter weird liar orange decline behind total despair fly
Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex):  0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
Public key (SS58): 5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
Account ID:        0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
SS58 Address:      5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
```

> [!IMPORTANT]
> Run these once

```bash
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
code customSpec.json
```

Change line two to `"name": "My Custom Testnet",`

Paste the AURA keys you saved earlier.

```json
"aura": {
    "authorities": [
        "5G4VGku8FiTybCeoj6NkfmdJFD68U68EYfuW7Azwp7ULBLYq",
        "5H4MBk8xcXFHayugpCJxXfksbpc1A2wCs55xJENSEWTribVY"
    ]
},
```

Paste the GRANDPA keys you saved earlier.

```json
"grandpa": {
    "authorities": [
        [
            "5FvPQX7zaCbmRJz5ovpoaxfi1AwHWuDiikXSQFtNxvAoLAj2",
            1
        ],
        [
            "5Eo1JXh1MzordXj6Hjs9r9q9tYitH5xoLUnG2wjWf5Kc1y3Q",
            1
        ]
    ]
},
```

Create a customSpecRaw.json file by running the following command.

```bash
./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
```

### Run Node01

```bash
./target/release/node-template \
  --base-path /tmp/node01 \
  --chain ./customSpecRaw.json \
  --port 30333 \
  --rpc-port 9945 \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
  --validator \
  --rpc-methods Unsafe \
  --name MyNode01 \
  --password-interactive
```

If it's run stop with `CTRL + C`

Run these commands for Node01. Paste your secret seed and password when prompted.

```bash
./target/release/node-template key insert --base-path /tmp/node01 \
  --chain customSpecRaw.json \
  --scheme Sr25519 \
  --suri <your-secret-seed> \
  --password-interactive \
  --key-type aura
```

```bash
./target/release/node-template key insert \
  --base-path /tmp/node01 \
  --chain customSpecRaw.json \
  --scheme Ed25519 \
  --suri <your-secret-key> \
  --password-interactive \
  --key-type gran
```

```bash
ls /tmp/node01/chains/local_testnet/keystore
```

Run the above command, the output will be as follows

```bash
617572617e016f19ab623ba5f487f540017c1edbab06c0b211a16d40531dbd62d94ceb24
6772616e4ac976937e53fd836512cfd288bb438584ba366cbf9be403a0acd82c1c7c0739
```

```bash
./target/release/node-template \
  --base-path /tmp/node01 \
  --chain ./customSpecRaw.json \
  --port 30333 \
  --rpc-port 9945 \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
  --validator \
  --rpc-methods Unsafe \
  --name MyNode01 \
  --password-interactive
```

### Add Participants - Node02

```bash
./target/release/node-template \
  --base-path /tmp/node02 \
  --chain ./customSpecRaw.json \
  --port 30334 \
  --rpc-port 9946 \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
  --validator \
  --rpc-methods Unsafe \
  --name MyNode02 \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
  --password-interactive
```

If it's run stop with `CTRL + C`

```bash
./target/release/node-template key insert --base-path /tmp/node02 \
  --chain customSpecRaw.json \
  --scheme Sr25519 \
  --suri <second-participant-secret-seed> \
  --password-interactive \
  --key-type aura
```

```bash
./target/release/node-template key insert --base-path /tmp/node02 \
  --chain customSpecRaw.json \
  --scheme Ed25519 --suri <second-participant-secret-seed> \
  --password-interactive \
  --key-type gran
```

```bash
ls /tmp/node02/chains/local_testnet/keystore
```

Run the above command, the output will be as follows

```bash
617572610a6cadb3d6f55a121de4c89754dd835e634ae83249734dfad01c2fae7e9ac102
6772616e5f273f61a4910897cec969b598a70a832fb7894ad7c741e2a559617898426f20
```

When you run the node01, you will see the following output.

```bash
🏷 Local node identity is: 12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX
```

```bash
./target/release/node-template \
  --base-path /tmp/node02 \
  --chain ./customSpecRaw.json \
  --port 30334 \
  --rpc-port 9946 \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 0' \
  --validator \
  --rpc-methods Unsafe \
  --name MyNode02 \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/<your-node01-identity> \
  --password-interactive
```

## Smart contracts

[Substrate Documentation](https://docs.substrate.io/tutorials/smart-contracts/)


### Update Rust Toolchain

```bash
rustup component add rust-src
rustup target add wasm32-unknown-unknown --toolchain nightly
```

### Install cargo-contract CLI Tools

```bash
rustup component add rust-src
cargo install --force --locked cargo-contract --version 2.0.0-rc
```

### Install the Substrate Contracts Node

```bash
cargo install contracts-node --git https://github.com/paritytech/substrate-contracts-node.git --tag <latest-tag> --force --locked
```

### Smart Contract

#### Create a Smart Contract

```bash
cargo contract new flipper
```

#### Test The Contract

```bash
cargo test
```

#### Build The Contract

```bash
cargo contract build
```

### Start the Substrate Contracts Node

```bash
substrate-contracts-node --log info,runtime::contracts=debug 2>&1
```

### Deploy the contract

#### Uploading the ink! Contract Code

You should save the contract address to a variable for later use. `Contract 5GRAVvuSXx8pCpRUDHzK6S1r2FjadahRQ6NEgAVooQ2bB8r5`

```bash
cargo contract instantiate --constructor new --args "false" --suri //Alice --salt $(date +%s)
```

```bash
cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS --message get --suri //Alice --dry-run

or

cargo contract call --contract <INSTANTIATED_CONTRACT_ADDRESS> --message get --suri //Alice --dry-run
```

```bash
cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS --message flip --suri //Alice

or 

cargo contract call --contract <INSTANTIATED_CONTRACT_ADDRESS> --message flip --suri //Alice
```
