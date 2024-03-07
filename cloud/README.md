# Cloud Infrastructure
EOS EVM public endpoint cloud infrastructure documentation.

> [!CAUTION]
> This repo is **public**, do not document [sensitive information](https://github.com/eosnetworkfoundation/engineering/blob/main/standards/secrets.md) here!

> [!IMPORTANT]
> > As an open-source software organization funded by and with obligations to our community, we make as much information publicly available as possible. However, [sensitive details](https://github.com/eosnetworkfoundation/engineering/blob/main/standards/secrets.md) are described using labels that are [distinct](https://en.wiktionary.org/wiki/distinct) and [definite](https://en.wiktionary.org/wiki/definite) without being [determinate](https://en.wiktionary.org/wiki/determinate). Documentation in the private [eos-evm-internal](https://github.com/eosnetworkfoundation/eos-evm-internal/tree/main/cloud) repo maps the indeterminate labels to our implementation-specific details. All of these details would be different for anyone else deploying this software stack anyways.

### Index
1. [Endpoints](#endpoints)
    1. [Ownership](#ownership)
    1. [Endpoint Health Check](#endpoint-health-check)
1. [See Also](#see-also)

## Endpoints
The community maintains the following endpoints for the public to interact with the EOS EVM.

Endpoint | Mainnet | Testnet | Notes
--- | :---: | :---: | ---
API | `api.evm.eosnetwork.com` | `api.testnet.evm.eosnetwork.com` | RPC API for tools like [Frame](https://frame.sh), [MetaMask](https://metamask.io), and [Rabby](https://rabby.io) to interact with the EOS EVM without running a full node.
Bridge | [bridge.evm.eosnetwork.com](https://bridge.evm.eosnetwork.com) | [bridge.testnet.evm.eosnetwork.com](https://bridge.testnet.evm.eosnetwork.com) | Trustless bridge to move EOS tokens between the native chain and the EVM.
Explorer | [explorer.evm.eosnetwork.com](https://explorer.evm.eosnetwork.com) | [explorer.testnet.evm.eosnetwork.com](https://explorer.testnet.evm.eosnetwork.com) | Block explorer and transaction viewer, running a [fork](https://github.com/eosnetworkfoundation/blockscout) of [Blockscout](https://www.blockscout.com).
Faucet | - | [faucet.testnet.evm.eosnetwork.com](https://faucet.testnet.evm.eosnetwork.com) | Obtain EOS tokens for testing. The faucet is run by [EOS Nation](https://eosnation.io).

### Ownership
Ownership ultimately lies with the community, which chose to use on-chain consensus mechanisms to delegate a leadership role in EOS EVM core software development and public endpoint operations to the [EOS Network Foundation](https://eosnetwork.com). The ENF collaborates with community contributors such as [EOS Labs](https://www.eoslabs.io), [EOS Nation](https://eosnation.io), and independent contributors to accomplish these goals.

Responsibility for EOS EVM public endpoint operations is shared between several teams.
- **ENF Automation**
    - Amazon Web Services (AWS) accounts, identity, and access management (IAM).
    - Cloud network infrastructure for the API, bridge, and explorer.
    - Cost analysis for ENF infrastructure.
    - Domain Name Service (DNS) for all endpoints.
- **ENF Engineering**
    - Compute.
    - Core software development, including frontend.
    - Database management.
    - Deployment and upgrades of EVM core components.
- **ENF Operations**
    - Billing for the API, bridge, and explorer.
- **EOS Nation**
    - Faucet API, backend, billing, cloud infrastructure, and frontend.

### Endpoint Health Check
Tests used to determine if endpoints are healthy live [here](./endpoint-health-check.md).

## See Also
More resources.
- [`../README.md`](../README.md) ⤴
- [eos-evm-internal](https://github.com/eosnetworkfoundation/eos-evm-internal) - internal-facing documentation of a [sensitive](https://github.com/eosnetworkfoundation/engineering/blob/main/standards/secrets.md) nature.
- [Runbooks](../runbooks/README.md)

***
> **_Legal Notice_**  
> This repo contains assets created in collaboration with a large language model, machine learning algorithm, or weak artificial intelligence (AI). This notice is required in some countries.
