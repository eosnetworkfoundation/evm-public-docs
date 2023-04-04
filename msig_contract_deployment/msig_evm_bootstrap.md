<h1> Multi-sig evm account bootstrapping </h1>

<h2> Overview: </h2>

This document will describe the multisig bootstrapping process of EVM contract in mainnet, after the creation of account eosio.evm (see https://github.com/eosnetworkfoundation/evm-public-docs/blob/main/msig_contract_deployment/msig_account_creation.md for previous multisig for account creation details).

Current proposed EVM bootstrapping multisig on mainnet is: https://bloks.io/msig/admin1.enf/evmboot

The following actions are included in the above multisig:

- Action 1 & 2: set code & set ABI:

  the EVM contract code (with hash 885f087ffedf8b7163661430a5cf9cb8c9dd5ea0e137c806d50ca5bc7e4a1271) was compiled and verified in the following environment:

  - contract commit hash: f7583bf613635be75b79d4055a5660289136b09f (HEAD -> main, tag: v0.4.0-rc1)
  - clang version 11
  - cdt-cpp version 3.1.0
  
- Action 3: eosio.evm::init:
  
  This is the init action to initialize the smart contract, which contains the following pararmeters:
  - chainid: 17777, mainnet
  - gas price: 150000000000, which means 150 Gwei is the required gas price. transaction will fails if the gas price is less than 150 Gwei.
  - miner_cut: 10000, which means 10% of the gas fee go to the miner account (that offer CPU & NET resources, but not RAM). The rest 90% will go to eosio.evm to cover the RAM cost.
  - ingress_bridge_fee: 0.0100 EOS. a fixed fee of 0.01 EOS will be charged for each bridging action from EOS world to EVM world. so if and EOS account x transfer 1 EOS to EVM account b, b will get 0.99 EOS. This won't apply in the other direction (EVM->EOS).
  
- Action 4: eosio.evm::freeze with value = true

  This action is to temporarily freeze the smart contract, disallowing users to push any EVM actions, or transfer token from/to the EVM world via the bridge, or withdraw miner balance. We will freeze the contract until the main launch date in case any patches are required.
  
- Action 5 & 6: create eosio.evm::freeze permission and link the freeze action to eosio.evm::freeze permission. Please refer to the permission structure of eosio.evm as illustrated in https://github.com/eosnetworkfoundation/evm-public-docs/blob/main/msig_contract_deployment/msig_account_creation.md.

According to the current permission structure of eosio.evm::active (see https://bloks.io/account/eosio.evm#keys), we need 3 approvals from atticlabeosb, bp.defi, eosasia11111, eosnationftw or newdex.bp, plus the approval of enf to satisfy eosio.evm::active in order to execute this multisig proposal.


