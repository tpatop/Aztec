# Aztec Sequencer Node

## [Russian version](readme.ru.md)

## Server Requirements

**Minimum:**  
8 CPU, 16 GB RAM, 100+ SSD

### Server Rental

Pay for servers with crypto or Russian cards:  
[play2go](https://play2go.cloud/?ref_id=q7Of8tsD3Ko)  
[vdsiba](https://www.vdsina.com/?partner=n974g9fq23)

## Preparation Before Installation

### Funding Your Wallet

You will need a new EVM wallet with test ETH tokens on the Sepolia network. Use the [faucet](https://sepolia-faucet.pk910.de/), but note that the required amount is currently quite high due to the validator registration quota (~50 ETH).

### RPC

You need an Ethereum Sepolia RPC and Beacon endpoint for the node to work. It is recommended to deploy your own service or buy a private RPC (I can provide one if needed).

## Node Installation

1. Open your terminal and connect to your server via SSH.
2. Update system components and install required utilities:

    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```

    ```bash
    sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
    ```

3. Install Aztec tools, press `y` twice when prompted:

    ```bash
    bash -i <(curl -s https://install.aztec.network/)
    ```

4. Update your environment and check the tools:

    ```bash
    source ~/.bash_profile
    ```
    ```bash
    aztec-up alpha-testnet
    ```

5. Create files to manage your node:  
    Create a directory and enter it:
    ```bash
    mkdir -p aztec && cd aztec
    ```
    Create a launch file:
    ```bash
    nano docker-compose.yml
    ```
    Paste the following snippet, change ports if needed:
    ```
    services:
      node:
        image: aztecprotocol/aztec:0.87.2
        environment:
          ETHEREUM_HOSTS: "${ETHEREUM_HOSTS}"
          L1_CONSENSUS_HOST_URLS: "${L1_CONSENSUS_HOST_URLS}"
          DATA_DIRECTORY: /data
          VALIDATOR_PRIVATE_KEY: "${VALIDATOR_PRIVATE_KEY}"
          P2P_IP: "${P2P_IP}"
          PXE_PORT: "${PXE_PORT}"
          TXE_PORT: "${TXE_PORT}"
          LOG_LEVEL: debug
        entrypoint: >
          sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet start --node --archiver --sequencer'
        volumes:
          - /home/aztec/node:/data
        ports:
          - 40400:40400/tcp
          - 40400:40400/udp
          - 8080:8080
      ```
    Save with CTRL+S, CTRL+X

    Create a configuration file:
    ```bash
    nano .env
    ```
    Paste and fill in your values:
    ```
    L1_SEPOLIA_RPC_URL=<Your_Sepolia_RPC_URL>
    L1_BEACON_URLS=<Your_BEACON_URL>
    VALIDATOR_PRIVATE_KEY=<Your_Privat_Key_with_0x>
    P2P_IP=<Your_Server_IP>
    ```
    Save with CTRL+S, CTRL+X

6. Start the operator (node) and check logs:

    ```bash
    docker-compose up --build -d
    docker-compose logs -f --tail 20
    ```

## Getting the `Apprentice` Role in Discord

1. After your node syncs, get the latest synced block number:

    ```bash
    curl -s -X POST -H 'Content-Type: application/json' \
    -d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
    http://localhost:8080 | jq -r ".result.proven.number"
    ```

2. In the command below, replace `<BLOCK_NUMBER>` with your block number and port 8080 if you changed it:

    ```bash
    curl -s -X POST -H 'Content-Type: application/json' \
    -d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["<BLOCK_NUMBER>","<BLOCK_NUMBER>"],"id":67}' \
    http://localhost:8080 | jq -r ".result"
    ```

3. In the [project Discord](https://discord.gg/aztec), in the `#operators | start-here` channel, use the `/operator start` command and provide the required data.

## Validator Registration

You need to send a transaction at a specific time and hope for luck, as the number of applicants is growing and the daily quota is only 5.

Replace the data and run the command:
```bash
aztec add-l1-validator \
--l1-rpc-urls <Your_Sepolia_RPC_URL> \
--private-key <Your_Private_Key> \
--attester <Your_Wallet_Address> \
--proposer-eoa <Your_Wallet_Address> \
--staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
--l1-chain-id 11155111
