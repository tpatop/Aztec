# Aztec Sequencer Node

## [English version](#readme.md)

## ⚠️ Требования к серверу

**Минимальные:**  
8 CPU, 16 GB RAM, 100+ SSD

### Аренда серверов

Оплата серверов криптой и российскими картами: 
[play2go](https://play2go.cloud/?ref_id=q7Of8tsD3Ko)
[vdsiba](https://www.vdsina.com/?partner=n974g9fq23)


## ⚠️ Подготовка к установке

### Пополнение кошелька

Для установки потребуется новый EVM кошелек с тестовыми токенами ETH в сети Sepolia. В [кране](https://sepolia-faucet.pk910.de/), однако количество необходимых токенов на данный момент весьма высокое из-за квоты на регистрацию валидаторов (~50 eth).

### RPC

Для работы ноды потребуется RPC в сети Ethereum Sepolia и Beacon. Советую развернуть свой сервис или приобрести частный rpc (Готов предоставить)

## 🛠️ Установка ноды

1. Заходим в терминал, подключаемся к серверу по SSH.
2. Обновим компоненты и установим необходимые утилиты:

    ```bash
    sudo apt-get update && sudo apt-get upgrade -y
    ```

    ```bash
    sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev  -y
    ```

3. Установи инструменты Aztec, два раза нажми `y`:

    ```bash
    bash -i <(curl -s https://install.aztec.network/)
    ```

4. Обнови окружение и проверь работу инструментов:

    ```bash
    source ~/.bash_profile
    ```
    ```bash
    aztec-up alpha-testnet
    ```

5. Создай файл для управления нодой:
    Создай директорию и перейди в неё
    ```bash
    mkdir -p aztec && cd aztec
    ```
    Создай файл для запуска
    ```bash
    nano docker-compose.yml
    ```
    Вставь в файл следующий фрагмент, при необходимости замени порты
    ```
    services:
        node:
            image: aztecprotocol/aztec:0.85.0-alpha-testnet.5
            environment:
            ETHEREUM_HOSTS: "${L1_SEPOLIA_RPC_URL}"
            L1_CONSENSUS_HOST_URLS: "${L1_BEACON_URLS}"
            DATA_DIRECTORY: /data
            VALIDATOR_PRIVATE_KEY: "${VALIDATOR_PRIVATE_KEY}"
            P2P_IP: "${P2P_IP}"
            PXE_PORT: "${PXE_PORT}"
            TXE_PORT: "${TXE_PORT}"
            LOG_LEVEL: debug
            entrypoint: >
            sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet start --node --archiver --sequencer'
            volumes:
            - /home/my-node/node:/data
            ports:
            - 40400:40400/tcp
            - 40400:40400/udp
            - 8080:8080
    ```
    Сохрани CTRL+S, CTRL+X

    Создай файл с конфигурацией
    ```bash
    nano .env
    ```
    Вставь в файл, заменив на свои значения
    ```
    L1_SEPOLIA_RPC_URL=<Your_Sepolia_RPC_URL>
    L1_BEACON_URLS=<Your_BEACON_URL>
    VALIDATOR_PRIVATE_KEY=<Your_Privat_Key_with_0x>
    P2P_IP=<Your_Server_IP>
    ```
    Сохрани CTRL+S, CTRL+X

6. Запусти оператора (ноду) и проверь логи:

    ```bash
    docker-compose up --build -d
    docker-compose logs -f --tail 20
    ```

## Получение роли `Apprentice` в Discord

1. После синхронизации ноды получи номер последнего синхронизированного блока:

    ```bash
    curl -s -X POST -H 'Content-Type: application/json' \
    -d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
    http://localhost:8080 | jq -r ".result.proven.number"
    ```

2. В команде ниже замени `<BLOCK_NUMBER>` на номер своего блока, и порт 8080 на свой:

    ```bash
    curl -s -X POST -H 'Content-Type: application/json' \
    -d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["<BLOCK_NUMBER>","<BLOCK_NUMBER>"],"id":67}' \
    http://localhost:8080 | jq -r ".result"
    ```

3. В [дискорде проекта](https://discord.gg/aztec), в ветке `#operators | start-here` введи команду `/operator start`, вставь необходимые данные.


## Регистрация валидатора

Необходимо отправить транзакцию в определенное время и верить в удачу, ибо количество желающих с каждым днем всё больше, а суточная квота всего 5.

Замени данные и выполняй команду
```bash
aztec add-l1-validator \
--l1-rpc-urls <Your_Sepolia_RPC_URL> \
--private-key <Your_Private_Key> \
--attester <Your_Wallet_Address> \
--proposer-eoa <Your_Wallet_Address> \
--staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
--l1-chain-id 11155111