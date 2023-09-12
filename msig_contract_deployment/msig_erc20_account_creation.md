# Multi-sig account creation of evm accounts for EOS-ERC20 token bridge

## Overview 
EOS-EVM will soon support trustless token bridge between tokens on native EOS (for example: USDT on EOS) and ERC-20 compatible tokens on EOS-EVM. (contract implementation can be found in https://github.com/eosnetworkfoundation/evm-bridge-contracts). We need to create the account ```eosio.erc2o``` and ```eosio.evmin``` within this paragraph.

### account eosio.erc2o

  it is the account that handles the main logic of token bridge operations, such as token registration, set ingress/egress fee, token transfer between native EOS and EOS-EVM, withdrawfee, etc. The EVM ERC-20 bridge contract will be deployed into this account.

**permission structure:**
  - owner: set to eosio@active
  - active: set to eosio.evm@active

### account eosio.evmin

  it is the account that acts as a proxy in the EOS->EVM flow to receive tokens and redirect received tokens to proper accounts.

**permission structure:**
  - owner: set to eosio@active
  - active: set to eosio.evm@active
