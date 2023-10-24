# awm-relayer

Reference relayer implementation for cross-chain Avalanche Warp Message delivery.

## Usage

### Building

Build the relayer by running the script:

```bash
./scripts/build.sh
```

Build a Docker image by running the script:
```
./scripts/build-local-image.sh
```

### Running

The relayer binary accepts a path to a JSON configuration file as the sole argument. Command line configuration arguments are not currently supported.

```bash
./build/awm-relayer --config-file path-to-config
```

### Configuration

The relayer is configured via a JSON file, the path to which is passed in via the `--config-file` command line argument. The following configuration options are available:

`"log-level": "debug" | "info" | "warn" | "error" | "fatal" | "panic"` 
- The log level for the relayer. Defaults to `info`.

`"network-id": integer`
- The ID of the Avalanche network to which the relayer will connect. Defaults to `1` (Mainnet).

`"p-chain-api-url": string`
- The URL of the Avalanche P-Chain API node to which the relayer will connect. Defaults to `https://api.avax.network`.

`"encrypt-connection": boolean`
- Whether or not to encrypt the connection to the P-Chain API node. Defaults to `true`.

`"storage-location": string`
- The path to the directory in which the relayer will store its state. Defaults to `./awm-relayer-storage`.

`"source-subnets": []SourceSubnets`
- The list of source subnets to support. Each `SourceSubnet` has the following configuration:

  `"subnet-id": string` 
  - cb58-encoded Subnet ID

  `"blockchain-id": string` 
  - cb58-encoded Blockchain ID

  `"vm": string` 
  - The VM type of the source subnet.

  `"api-node-host": string` 
  - The host of the source subnet's API node.

  `"api-node-port": integer` 
  - The port of the source subnet's API node.

  `"encrypt-connection": boolean` 
  - Whether or not to encrypt the connection to the source subnet's API node.

  `"rpc-endpoint": string` 
  - The RPC endpoint of the source subnet's API node. Used in favor of `api-node-host`, `api-node-port`, and `encrypt-connection` when constructing the endpoint

  `"ws-endpoint": string` 
  - The WebSocket endpoint of the source subnet's API node. Used in favor of `api-node-host`, `api-node-port`, and `encrypt-connection` when constructing the endpoint

  `"message-contracts": map[string]MessageProtocolConfig` 
  - Map of contract addresses to the config options of the protocol at that address. Each `MessageProtocolConfig` consists of a unique `message-format` name, and the raw JSON `settings`

  `"supported-destinations": []string` 
  - List of destination subnet IDs that the source subnet supports. If empty, then all destinations are supported.

`"destination-subnets": []DestinationSubnets`
- The list of destination subnets to support. Each `DestinationSubnet` has the following configuration:

  `"subnet-id": string`
  - cb58-encoded Subnet ID

  `"blockchain-id": string` 
  - cb58-encoded Blockchain ID

  `"vm": string` 
  - The VM type of the source subnet.

  `"api-node-host": string` 
  - The host of the source subnet's API node.

  `"api-node-port": integer` 
  - The port of the source subnet's API node.

  `"encrypt-connection": boolean` 
  - Whether or not to encrypt the connection to the source subnet's API node.

  `"rpc-endpoint": string` 
  - The RPC endpoint of the destination subnet's API node. Used in favor of `api-node-host`, `api-node-port`, and `encrypt-connection` when constructing the endpoint

  `"account-private-key": string` 
  - The hex-encoded private key to use for signing transactions on the destination subnet. May be provided by the environment variable `ACCOUNT_PRIVATE_KEY`. Each `destination-subnet` may use a separate private key by appending the blockchain ID to the private key environment variable name, e.g. `ACCOUNT_PRIVATE_KEY_11111111111111111111111111111111LpoYY` 

## Architecture

### Components

The relayer consists of the following components:

- At the global level:
    - *P2P app network*: issues signature `AppRequests`
    - *P-Chain client*: gets the validators for a subnet
    - *JSON database*: stores latest processed block for each source subnet
- Per Source subnet
    - *Subscriber*: listens for logs pertaining to cross-chain message transactions
    - *Source RPC client*: queries for missed blocks on startup
- Per Destination subnet
    - *Destination RPC client*: broadcasts transactions to the destination

### Data flow

<div align="center">
  <img src="resources/relayer-diagram.png?raw=true">
</div>

## Testing

### Unit tests

Unit tests can be ran locally by running the command in the root of the project:

```bash
./scripts/test.sh
```

### E2E tests

E2E tests are ran as part of CI, but can also be ran locally with the `--local` flag. To run the E2E tests locally, you'll need to install Gingko following the intructions [here](https://onsi.github.io/ginkgo/#installing-ginkgo)

Next, provide the path to the `subnet-evm` repository and the path to a writeable data directory (this example uses `~/subnet-evm` and `~/tmp/e2e-test`) to use for the tests:
```bash
./scripts/e2e_test.sh --local --subnet-evm ~/subnet-evm --data-dir ~/tmp/e2e-test
```
### Generate Mocks

[Gomock](https://pkg.go.dev/go.uber.org/mock/gomock) is used to generate mocks for testing. To generate mocks, run the following command at the root of the project:

```bash
go generate ./...
```
