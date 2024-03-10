# Cloud Infrastructure
EOS EVM public endpoint cloud infrastructure documentation.

> [!CAUTION]
> This repo is **public**, do not document [sensitive information](https://github.com/eosnetworkfoundation/engineering/blob/main/standards/secrets.md) here!

> [!IMPORTANT]
> > As an open-source software organization funded by and with obligations to our community, we make as much information publicly available as possible. However, [sensitive details](https://github.com/eosnetworkfoundation/engineering/blob/main/standards/secrets.md) are described using labels that are [distinct](https://en.wiktionary.org/wiki/distinct) and [definite](https://en.wiktionary.org/wiki/definite) without being [determinate](https://en.wiktionary.org/wiki/determinate). Documentation in the private [eos-evm-internal](https://github.com/eosnetworkfoundation/eos-evm-internal/tree/main/cloud) repo maps the indeterminate labels to our implementation-specific details. All of these details would be different for anyone else deploying this software stack anyways.

<!-- contents box begin -->
<table>
<tr/>
<tr>
<td width="200">
<p/>
<div align="center">
<b>Contents</b>
</div>
<p/>
<!-- contents markdown begin -->

1. [Endpoints](#endpoints)
1. [Ownership](#ownership)
1. [Context](#context)
    1. [Environments](#environments)
    1. [Datacenters](#datacenters)
    1. [Resources](#resources)
        1. [Names](#names)
        1. [Tags](#tags)
1. [System Architecture](#system-architecture)
1. [Deployment Strategy](#deployment-strategy)
1. [See Also](#see-also)

<!-- contents markdown end -->
<p/>
</td>
</tr>
</table>
<!-- contents box end -->

## Endpoints
The community maintains the following endpoints for the public to interact with the EOS EVM.

Endpoint | Mainnet | Testnet | Notes
--- | :---: | :---: | ---
API | `api.evm.eosnetwork.com` | `api.testnet.evm.eosnetwork.com` | RPC API for tools like [Frame](https://frame.sh), [MetaMask](https://metamask.io), and [Rabby](https://rabby.io) to interact with the EOS EVM without running a full node.
Bridge | [bridge.evm.eosnetwork.com](https://bridge.evm.eosnetwork.com) | [bridge.testnet.evm.eosnetwork.com](https://bridge.testnet.evm.eosnetwork.com) | Trustless bridge to move EOS tokens between the native chain and the EVM.
Explorer | [explorer.evm.eosnetwork.com](https://explorer.evm.eosnetwork.com) | [explorer.testnet.evm.eosnetwork.com](https://explorer.testnet.evm.eosnetwork.com) | Block explorer and transaction viewer, running a [fork](https://github.com/eosnetworkfoundation/blockscout) of [Blockscout](https://www.blockscout.com).
Faucet | - | [faucet.testnet.evm.eosnetwork.com](https://faucet.testnet.evm.eosnetwork.com) | Obtain EOS tokens for testing. The faucet is run by [EOS Nation](https://eosnation.io).

## Ownership
Ownership ultimately lies with the community, which chose to use on-chain consensus mechanisms to delegate a leadership role over EOS EVM core software development and public endpoint operations to the [EOS Network Foundation](https://eosnetwork.com). The ENF collaborates with community contributors such as [EOS Labs](https://www.eoslabs.io), [EOS Nation](https://eosnation.io), and independent contributors to accomplish these goals.

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

> [!NOTE]
> > **2024-03-07**  
> > EOS Labs recently volunteered to run the public endpoints. That means they will become responsible for all list elements above, except for core software development and probably the faucet.

## Context
The EOS EVM infrastructure is hosted on Amazon Web Services (AWS) and deployed manually.

### Environments
There are currently two environments, a staging environment using the testnet chain and a production environment using the mainnet chain. Each environment is deployed to a different AWS account.

Environment | Chain | AWS Account
--- | --- | ---
Production | EOS EVM Mainnet | `evm-mainnet`
Staging | EOS EVM Testnet | `evm-testnet`

The cloud network infrastructure is intentionally kept identical between all environments to increase the likelihood that bugs are discovered before changes are deployed to production.

### Datacenters
Each environment spans multiple [AWS regions](https://aws.amazon.com/about-aws/global-infrastructure/regions_az), which are helpful to think of as datacenters.

Name | Region
--- | ---
`ap` | Asia-Pacific
`us` | United States

All systems use multiple availability zones (AZs) within each region, where applicable.

> [!TIP]
> > Globally distributed datacenters minimize the latency to users and maximize fault tolerance. Catastrophic failure of multiple availability zones in a single region is possible, both [on accident](https://www.theregister.com/2023/04/26/google_cloud_outage) and [on purpose](https://www.wired.com/story/far-right-extremist-allegedly-plotted-blow-up-amazon-data-centers).

### Resources
AWS supports user-defined names and tags to help identify resources.

#### Names
Resource names are intended to be unique and specific enough that they are unambiguous, without being so specific that they aren't safe to discuss in an open forum. The naming schema is...
```js
// AWS account name schema
account = `${product || repo}-${environment}`
// AWS resource name schema
resource = `${account}-${datacenter}-${system}-${service}-${component}-${version}`
```
...where:
- **component** - (optional) friendly name used to differentiate components, such as one virtual machine (VM) running a database and another VM running a web server.
- **datacenter** - documented [above](#datacenters).
- **environment** - explained [above](#environments).
- **service** - shorthand for the [AWS service](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services) containing this resource.
- **system** - the larger system or deployment this resource is a part of.
- **version** - (optional) semantic version of software deployed to this resource, used only when resources are deployed concurrently with different versions.

Here are some examples.
```
evm-mainnet-ap-api-vm-miner-v0.1.1
evm-testnet-us-explorer-lb
```

#### Tags
In addition to the default tags populated by AWS, resources are tagged with the following to provide traceability.

Tag | Type | Deployment | Description
--- | :---: | :---: | ---
`billing-use` | Enum | All | Used for cost analysis in the management account (e.g. `evm-api`).
`branch` | String | Automated | The `git` branch containing the code for this resource, if any.
`build` | URL | Automated | The URL of the CI/CD build that deployed this resource.
`commit` | SHA-1 | Automated | The `git` commit containing the code for this resource, if any.
`email` | Email | All | The email address of the individual who deployed this resource.
`env` | Enum | All | The environment this resource belongs to (`prod`, `staging`, `dev`, etc.).
`manual` | Boolean | All | Whether this resource was deployed manually or by an automated system.
`repo` | URL | Any | The URL of the GitHub repository containing the code for this resource, if any.
`tag` | String | Any | The `git` tag containing the code for this resource, if any.
`ticket` | URL | Manual | The ticket authorizing this resource to be deployed.

These tags can also be used as dimensions in the AWS cost analysis tool.

## System Architecture
Each [environment](#environments) contains the following systems.

System | Architecture | Notes
--- | :---: | ---
API | Web Application
Bridge | Web Application
Explorer | Web Application
Faucet | External System | Testnet only.
Metrics | AWS CloudWatch
Notifications | Event Handler

The web applications are all deployed almost the exact same way using the exact same components, so the web application architecture will be documented once and any system-specific deviations will be described along the way.

## Deployment Strategy
Infrastructure changes are **always** deployed, _one at a time_, as follows.
1. A maintenance window is scheduled with stakeholders, during which no other changes are taking place.
    - This guarantees all stakeholders are informed.
    - This reduces the number of independent variables, minimizing the time to resolution should service degradation be observed.
1. Testnet endpoint functionality is verified using a virtual private network (VPN) to perform [smoke tests](../runbooks/endpoint-smoke-test.md) against all affected endpoints, each from a number of different cities.
    - The cities selected must exercise all datacenters.
    - The set of cities should be large, to exercise content delivery networks (CDNs) or other edge compute.
    - The cities used and results observed must be written down so the tests can be reproduced.
    - If any tests fail then the deployment must be deferred until the system is in a known-good state.
1. Changes are deployed to the testnet staging environment.
1. Testnet endpoint functionality is validated using [smoke tests](../runbooks/endpoint-smoke-test.md) from the same cities as before.
1. A waiting period is observed.
    - This gives the community time to identify and report bugs.
    - This should be two business days to one week, and must be no less than twenty four (24) hours.
1. Mainnet endpoint functionality is verified using [smoke tests](../runbooks/endpoint-smoke-test.md) from a set of cities meeting the criteria above.
1. Changes are deployed to the mainnet production environment.
1. Mainnet endpoint functionality is validated using [smoke tests](../runbooks/endpoint-smoke-test.md) from the same cities as before.

If service degradation is observed at any point in this process then all changes must be reverted, and the process must start over.

## See Also
More resources.
- [`../README.md`](../README.md) ⤴
- [eos-evm-internal](https://github.com/eosnetworkfoundation/eos-evm-internal) - internal-facing documentation of a [sensitive](https://github.com/eosnetworkfoundation/engineering/blob/main/standards/secrets.md) nature.
- [Runbooks](../runbooks/README.md)

***
> **_Legal Notice_**  
> This repo contains assets created in collaboration with a large language model, machine learning algorithm, or weak artificial intelligence (AI). This notice is required in some countries.
