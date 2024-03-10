# Endpoint Smoke Test
This documents the process to validate a given EOS-EVM network endpoint is online, functional, and qualified to receive public traffic.

<!-- contents box begin -->
<table>
<tr/>
<tr>
<td width="225">
<p/>
<div align="center">
<b>Contents</b>
</div>
<p/>
<!-- contents markdown begin -->

1. [Bridge](#bridge)
1. [Explorer](#explorer)
1. [Faucet](#faucet)
1. [RPC API](#rpc-api)
1. [See Also](#see-also)

<!-- contents markdown end -->
<p/>
</td>
</tr>
</table>
<!-- contents box end -->

## Bridge
Smoke tests for the bridge.
1. Navigate to the bridge in your web browser and verify it loads.
    - [bridge.evm.eosnetwork.com](https://bridge.evm.eosnetwork.com)
    - [bridge.testnet.evm.eosnetwork.com](https://bridge.testnet.evm.eosnetwork.com)
1. Verify the endpoint redirects your web browser from HTTP to HTTPS when attempting to load it unencrypted.
    - [bridge.evm.eosnetwork.com](http://bridge.evm.eosnetwork.com)
    - [bridge.testnet.evm.eosnetwork.com](http://bridge.testnet.evm.eosnetwork.com)

## Explorer
Smoke tests for the explorer.
1. Navigate to the explorer in your web browser and verify it loads.
    - [explorer.evm.eosnetwork.com](https://explorer.evm.eosnetwork.com)
    - [explorer.testnet.evm.eosnetwork.com](https://explorer.testnet.evm.eosnetwork.com)
1. Verify the endpoint redirects your web browser from HTTP to HTTPS when attempting to load it unencrypted.
    - [explorer.evm.eosnetwork.com](http://explorer.evm.eosnetwork.com)
    - [explorer.testnet.evm.eosnetwork.com](http://explorer.testnet.evm.eosnetwork.com)

## Faucet
Smoke tests for the faucet.
1. Navigate to the faucet in your web browser and verify it loads.
    - [faucet.testnet.evm.eosnetwork.com](https://faucet.testnet.evm.eosnetwork.com)
1. Verify the endpoint redirects your web browser from HTTP to HTTPS when attempting to load it unencrypted.
    - [faucet.testnet.evm.eosnetwork.com](http://faucet.testnet.evm.eosnetwork.com)

## RPC API
Smoke tests for the RPC API.
1. Verify the RPC API returns the head block.
    - Mainnet
      ```bash
      curl -fsSL 'https://api.evm.eosnetwork.com' -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq .
      ```
      You should see something like this, where the `result` field will vary by incrementing once per second.
      ```json
      {
        "id": 1,
        "jsonrpc": "2.0",
        "result": "0x39bdd8"
      }
      ```
    - Testnet
      ```bash
      curl -fsSL 'https://api.testnet.evm.eosnetwork.com' -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq .
      ```
      You should see something like this, where the `result` field will vary by incrementing once per second.
      ```json
      {
        "id": 1,
        "jsonrpc": "2.0",
        "result": "0x39bdd8"
      }
      ```
1. Verify the RPC API rejects unencrypted requests.
    - Mainnet
      ```bash
      curl -fsSL 'http://api.evm.eosnetwork.com' -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq .
      ```
      Currently, we expect you to get a 500 response.
      ```
      curl: (22) The requested URL returned error: 500
      ```
    - Testnet
      ```bash
      curl -fsSL 'http://api.testnet.evm.eosnetwork.com' -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' | jq .
      ```
      Currently, we expect you to get a 500 response.
      ```
      curl: (22) The requested URL returned error: 500
      ```

## See Also
- [Endpoint Health Checks](../endpoint-health-checks.md)
- [eos-evm](https://github.com/eosnetworkfoundation/eos-evm) - core EOS Ethereum virtual machine source code
