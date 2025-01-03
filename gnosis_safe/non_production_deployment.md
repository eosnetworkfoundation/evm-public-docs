# Safe Global Deployment

**NOTICE: This deployment is not production ready**

#### Deploying contracts

Make sure that the Web3 API supports batch requests with an `eth_gasPrice` call as one of the requests in the batch. See [here](https://github.com/eosnetworkfoundation/eos-evm-node/pull/340).

Add a new request in the [safe-singleton-factory](https://github.com/safe-global/safe-singleton-factory/issues/new?assignees=&labels=new-chain&projects=&template=new_chain.yml&title=%5BNew+chain%5D%3A+) repo to support `exSat Mainnet` and follow the instructions in the issue template. See [exSat testnet request](https://github.com/safe-global/safe-singleton-factory/issues/817) as an example.

After the `safe-singleton-factory` PR gets merged in clone [safe-smart-account](https://github.com/safe-global/safe-smart-account) repo and checkout the latest release tag.

Modify `package.json` to point to the master branch of `safe-singleton-factory` repo.

```diff
-        "@safe-global/safe-singleton-factory": "^1.0.33",
+        "@safe-global/safe-singleton-factory": "github:safe-global/safe-singleton-factory#main",
```

Set `MNEMONIC` in .env
Edit `hardhat.config.ts` add `exSat Mainnet` as a new network.

```diff=
         zkSyncSepolia: {
             ...sharedNetworkConfig,
             url: "https://sepolia.era.zksync.dev",
             ethNetwork: "goerli",
             zksync: true,
             verifyURL: "https://explorer.sepolia.era.zksync.dev/contract_verification",
         },
+         exSat: {
+             ...sharedNetworkConfig,
+             url: "https://evm.exsat.network/",
+         },
      },
      deterministicDeployment,
```

Fund with some native tokens the first account represented by `MNEMONIC`.

Deploy the contracts

```bash=
npm install
npm run build
npx hardhat --network exSat deploy
```

After the deployment completes, a list with the transaction id of each deployed contract will be returned by the script.

```bash=
deploying "SimulateTxAccessor" (tx: 0xaf817b0ed57766ad76b3447c90464e13f2a9bd00d9391d1e3a4068c6fb972bb0)...: deployed at 0x3d4BA2E0884aa488718476ca2FB8Efc291A46199 with 237931 gas
deploying "SafeProxyFactory" (tx: 0x613d366baf4c225487726f574c14cb4547aadc49b7c3c8b5938bb8bdfefdc6c9)...: deployed at 0x4e1DCf7AD4e460CfD30791CCC4F9c8a4f820ec67 with 712622 gas
deploying "TokenCallbackHandler" (tx: 0x6e83e11565c09eee9a266cb443ba2033492483d46c0f9919ceaf5c2be847a40a)...: deployed at 0xeDCF620325E82e3B9836eaaeFdc4283E99Dd7562 with 453406 gas
deploying "CompatibilityFallbackHandler" (tx: 0xf9db3daa4503ec99129d3ffe4a1665d58d9cabb1ac31f2d28dfd50a87b2dd753)...: deployed at 0xfd0732Dc9E303f09fCEf3a7388Ad10A83459Ec99 with 1270132 gas
deploying "CreateCall" (tx: 0x3849cd14354fa408a0dc424daee1d2b886b0b991e2edeba80e52c1064f017b07)...: deployed at0x9b35Af71d77eaf8d7e40252370304687390A1A52 with 290470 gas
deploying "MultiSend" (tx: 0x1a526f1903f6214f84c9cb835200cb15b2cb9eafea6e57d3075c7744f96381a7)...: deployed at 0x38869bf66a61cF6bDB996A6aE40D5853Fd43B526 with 190062 gas
deploying "MultiSendCallOnly" (tx: 0x59c782612dfc356d269e01ca1605c78f7264a93ea315337f24f06fa22bef754b)...: deployed at 0x9641d764fc13c8B624c04430C7356C1C7C8102e2 with 142150 gas
deploying "SignMessageLib" (tx: 0x2dcff33c3f41ece48d114cfc42f9c762fe3ba15b469aeee244c41ddce1d841c3)...: deployed at 0xd53cd0aB83D845Ac265BE939c57F53AD838012c9 with 262417 gas
deploying "SafeToL2Setup" (tx: 0x73ca6a63e8d3c8840db209fe93e7d7b18ee00255b6fca94f57d7ee359dd1534a)...: deployed at 0xBD89A1CE4DDe368FFAB0eC35506eEcE0b1fFdc54 with 230863 gas
deploying "Safe" (tx: 0x394d9e6612c1b25acf75f25ce9e637c975d1d056a74a7f49051abfd97cec592d)...: deployed at 0x41675C099F32341bf84BFc5382aF534df5C7461a with 5150072 gas
deploying "SafeL2" (tx: 0xeb7470d355ce3746fb8407f68a78988d6bfa475c87a6a73b00a36cc343db1f30)...: deployed at 0x29fcB43b46531BcA003ddC8FCB67FFE91900C762 with 5332531 gas
deploying "SafeToL2Migration" (tx: 0x89d136cf4258441b5822c82c86a31ea133a17656d394261f0e8d000d637c2d4c)...: deployed at 0xfF83F6335d8930cBad1c0D439A841f01888D9f69 with 1283078 gas
deploying "SafeMigration" (tx: 0x9089eb2877f32bdc235d0299befb001fee5a7a606ed912ca8b897ee35cb7bf98)...: deployed at 0x526643F69b81B008F46d95CD5ced5eC0edFFDaC6 with 512858 gas

```

#### Updating safe-eth-py

Add a new request in the [safe-eth-py](https://github.com/safe-global/safe-eth-py/issues/new?assignees=&labels=add-new-address&projects=&template=add_safe_address_new_chain.yml&title=%5BNew+chain%5D%3A+%7Bchain+name%7D) repo to support `exSat Mainnet` and follow the instructions in the issue template. See [exSat testnet request](https://github.com/safe-global/safe-eth-py/issues/1495) as an example.


Wait for the `Validation successful!✅` message on the issue and for the automatically created PR to be merged on main.

Clone the `safe-eth-py` repo, checkout the main branch, build the python package and publish to the pip repository.

#### Updating safe-deployments

Clone the [safe-deployments](https://github.com/safe-global/safe-deployments) repo and checkout the latest release tag.

Create a new branch and modify the following files to include the deployment information for `exSat Mainnet`. See [here](https://github.com/safe-global/safe-deployments/pull/905/files) for the `exSat Testnet` example.

* src/assets/v1.4.1/compatibility_fallback_handler.json
* src/assets/v1.4.1/create_call.json
* src/assets/v1.4.1/multi_send.json
* src/assets/v1.4.1/multi_send_call_only.json
* src/assets/v1.4.1/safe.json
* src/assets/v1.4.1/safe_l2.json
* src/assets/v1.4.1/safe_migration.json
* src/assets/v1.4.1/safe_proxy_factory.json
* src/assets/v1.4.1/safe_to_l2_migration.json
* src/assets/v1.4.1/safe_to_l2_setup.json
* src/assets/v1.4.1/sign_message_lib.json
* src/assets/v1.4.1/simulate_tx_accessor.json

commit, push and create a new PR.

Now we can wait for a new release of the `safe-deployments` from the safe-global team or build our own npm package and push it to the npm package repository.

_instruction on how to build and publish the npm package for the safe-deployments repo are skiped_

#### Use our safe-deployments and safe-eth-py packages

If we are not waiting for the updated releases of the safe-deployments and safe-eth-py, we need to update some projects that rely on them

###### protocol-kit (safe-global/safe-core-sdk)

This is the `@packages/protocol-kit` package located under the `packages/protocol-kit` of the safe-core-sdk repo.

Update safe-deployments to our npm package.

Build and publish the npm package.

###### safe-global/safe-wallet-web

Update safe-deployments to our npm package.

Update protocol-kit to our npm package. (the one that we build above)

Build docker image and publish to the docker repository.

###### safe-global/safe-client-gateway

Update safe-deployments to our npm package.

Build docker image and publish to the docker repository.

###### safe-global/safe-transaction-service

Update safe-eth-py to our pip package.

Build docker image and publish to the docker repository.

#### Deploy infrastructure

Clone the [safe-infrastructure](https://github.com/safe-global/safe-infrastructure) repo in the VM that will host the infrastructure.

Copy the `.env.sample` to `.env` and set the correct `RPC_NODE_URL`.

In `docker-compose.yaml`

 * Set `NEXT_PUBLIC_SAFE_VERSION=1.4.1`
 * Change the following docker images with the ones we built above
    * safeglobal/safe-wallet-web:${UI_VERSION}
    * safeglobal/safe-client-gateway-nest:${CGW_VERSION}
    * safeglobal/safe-transaction-service:${TXS_VERSION}
 * Add `processing` queue to the WORKER_QUEUES in `txs-worker-indexer`

    ```diff
    -      - WORKER_QUEUES=default,indexing
    +      - WORKER_QUEUES=default,indexing,processing
    ```

In `container_env_files/cgw.env` add:

```diff
+FINGERPRINT_ENCRYPTION_KEY=your-encryption-key
+JWT_ISSUER=your-jwt-issuer
+JWT_SECRET=your-jwt-secret
+PUSH_NOTIFICATIONS_API_SERVICE_ACCOUNT_CLIENT_EMAIL=valid-email@example.com
```

In `container_env_files/ui.env` set
`NEXT_PUBLIC_WC_PROJECT_ID` with the wallet connect project ID.

Launch infrastructure with 
```bash=
docker-compose up -d
```

#### Configure safe-config-service

Open http://1.2.3.4/cfg/admin/

Create a new chain for exSat Mainnet.
Besides the self descriptive fields, be sure to:
* Check L2 checkbox
* Add a new feature called **SAFE_141** and assign it to the newly created chain
* Use http://nginx:8000/txs in `Transaction service uri`
* Use **1.4.1** in `Recommended master copy version.


_Block explorer uri address template_
```
https://scan-testnet.exsat.network/address/{{address}}
```

_Block explorer uri tx hash template_
```
https://scan-testnet.exsat.network/tx/{{txHash}}
```
_Block explorer uri api template_
```
https://scan-testnet.exsat.network/api
```

#### Production deployment

For instruction on how to do a production deployment, complement these steps with the [following guide](https://github.com/safe-global/safe-infrastructure/blob/main/docs/running_production.md)
