
## Build Project
- Update Chain ID
    > Change `<chain_id>` below before running script. Example: `CHAIN_ID=osmosis-1`
    ```bash
    CHAIN_ID=<chain_id> && sed -i 's+$CHAIN_ID+'$CHAIN_ID'+g' docker-compose.yml
    ```
- Update Your Moniker
    > Change `<your_moniker>` below before running script.
    ```bash
    YOUR_MONIKER=<your_moniker> && sed -i 's+$YOUR_MONIKER+'$YOUR_MONIKER'+g' docker-compose.yml
    ```
- Add the `osmosis` repository as a git submodule
    ```bash
    git submodule add -f https://github.com/osmosis-labs/osmosis osmosis
    ```
- Checkout the most recent release
    ```bash
    cd osmosis && git checkout -f v4.1.0 && cd ..
    ```
    > Update the version above to reflect the latest release
- Build the containers
    ```bash
    docker-compose up -d --build
    ```

## Initialise Node
- Save the current chain id
    ```bash
    docker exec -it osmosis bash -c 'osmosisd config chain-id $CHAIN_ID'
    ```
- Intialise the osmosis working directory
    ```bash
    docker exec -it osmosis bash -c 'osmosisd init --chain-id=$CHAIN_ID $YOUR_MONIKER'
    ```
- Download the genesis file
    ```bash    
    docker exec -it osmosis bash -c 'curl https://media.githubusercontent.com/media/osmosis-labs/networks/main/osmosis-1/genesis.json > $DAEMON_HOME/config/genesis.json'
    ```
- Change ownership of the `osmosisd` path
    ```bash
    docker exec -it osmosis bash -c 'chown -R 1000:1000 $DAEMON_HOME'
    ```

## Update Peers and Min Gas Price
- Update `persistent_peers` in `.osmosisd/config/config.toml`
    > Current persistent peers can be obtaind from the [osmosis-1 peer list](https://github.com/osmosis-labs/networks/blob/main/peers.md)

    Example:
        ```bash
        # Comma separated list of nodes to keep persistent connections to
        persistent_peers = "7024d1ca024d5e33e7dc1dcb5ed08349768220b9@134.122.42.20:26656,d326ad6dffa7763853982f334022944259b4e7f4@143.110.212.33:26656,e437756a853061cc6f1639c2ac997d9f7e84be67@144.76.183.180:26656,64d36f3a186a113c02db0cf7c588c7c85d946b5b@209.97.132.170:26656,785bc83577e3980545bac051de8f57a9fd82695f@194.233.164.146:26656"
        ```
- Update minimum-gas-prices in `.osmosisd/config/app.toml`
    >  This prevents getting spammed by transactions with no gas.

    Example:
        ```bash
        # The minimum gas prices a validator is willing to accept for processing a
        # transaction. A transaction's fees must meet the minimum of any denomination
        # specified in this config (e.g. 0.25token1;0.0001token2).
        minimum-gas-prices = "0.001uosmo"
        ```

## Setup Cosmovisor
- Create directory structure for Cosmovisor
    ```bash
    docker exec -it osmosis bash -c 'mkdir -p $DAEMON_HOME/cosmovisor/{upgrades/v4/bin,genesis/bin}'
    ```
- Copy `osmosisd` install binary to `cosmovisor` bin
    ```bash
    docker exec -it osmosis bash -c 'cp $GOPATH/bin/osmosisd $DAEMON_HOME/cosmovisor/genesis/bin'
    ```
- Copy `osmosisd` build binary to `cosmovisor` upgrade bin
    ```bash
    docker exec -it osmosis bash -c 'cp build/osmosisd $DAEMON_HOME/cosmovisor/upgrades/v4/bin'
    ```
- Confirm `cosmovisor` version
    ```bash
    docker exec -it osmosis bash -c 'cosmovisor version'
    ```

## Helpful Snippets

- If you need to access the container directly use the following
    ```bash
    docker exec -it osmosis /bin/sh
    ```
- Check the status of your node
    ```bash
    docker exec -it osmosis bash -c 'osmosisd status'
    ```
- Start `osmosisd`
    ```bash
    docker exec -it osmosis bash -c 'osmosisd start'
    ```

## Inspiration
* [Setting Up a Genesis Osmosis Validator](https://github.com/osmosis-labs/networks/blob/main/genesis-validators.md)
* [Setting up a fullnode for Osmosis](https://catboss.medium.com/cat-boss-setting-up-a-fullnode-for-osmosis-osmosis-1-5f9752460f8f)
* [Turning a full node in to a validator node](https://catboss.medium.com/turning-a-full-node-in-to-a-validator-node-osmosis-1-36f3358f2412)
* [Install and setup Cosmovisor](https://github.com/osmosis-labs/networks/blob/main/osmosis-1/upgrades/cosmovisor.md)
* [Cosmosvisor](https://docs.cosmos.network/master/run-node/cosmovisor.html)
* [osmosis-1 peer list](https://github.com/osmosis-labs/networks/blob/main/peers.md)