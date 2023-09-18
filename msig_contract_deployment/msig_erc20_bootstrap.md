# Multi-sig ERC20 bridge contract bootstrapping

After the creation of ERC20 bridging accounts (`eosio.erc2o` & `eosio.evmin`), we need to bootstrap the bridging contracts:

## Multi-sig 1: Update the latest version of evm_runtime contract & withdraw funds

Action 1: Update the existing evm_runtime contract, adding supports for the following features required for ERC20 bridging:
  - *call action: this action will allow any account in the EOS side acts as the corresponding account (using deterministric reserverd address) in EVM side*
  - *admincall action: this action allow ENF & BPs to co-sign to satisfy eosio.evm@active and act on behalf of any EVM account in emergency situations.*
  - *event mechanisim: this is the mechanisim allowing event generated in the solidity contract in EVM to be sent as bridge message via inline action in native EOS.*

Action 2: withdraw 300 EOS from the eosio.evm's fee balance to enf for covering RAM and as ERC-20 bridge funds 

## Multi-sig 2: ERC-20 contract bootstrapping (depends on Multi-sig 1):

Action 1: Deploy core EVM ERC-20 contract to account `eosio.erc2o`. This contract contains the following actions:
  - *upgrade, upgradeto: set or deploy the ERC-20 upgradable implementation contract into EVM side and register the contract address in eosio.erc2o's multi_index table in EOS side.*
  - *regtoken: deploy the new proxy ERC-20 contract on EVM side (which will call the upgradable implementation contract) and setup the link between existing EOS tokens (such as USDT, IQ, ... etc) and the EVM proxy contract.*
  - *transfer notify handler (EOS->EVM flow implementation): action handler to call the EVM's ERC-20 token mint action (with the memo identifying the destination EVM address) via the eosio.evm contract, upon receiving the correct registered tokens from the EOS side.*
  - *onbridgemsg: event handler (EVM->EOS flow implmenetation): action hanlder to call the EOS inline transfer to transfer the registered token to the destination EOS account upon receiving the EVM event which is genereted from the implementation contract and sent via the eosio.evm runtime contract.*
  - *addregress, removeegress: similar to native EOS bridge, it controls the whilelist of EOS destinaction addresses with deployed code, to prevent misuse of CPU resource.*
  - *setegressfee, setingressfee: actions to control the bridge fee. Ingressfee is charged in form of the native tokens, while egressfee is charged in form on EOS.*
  - *withdrawfee: action to withdraw the accumulated ingress fee*
    
(sha256sum 86ae86e6bd6c3fff35a2dc7948626561ab0592c4a87fe2bed74d216079d90c7f  erc20.wasm)

Action 2: Deplay abi file of core EVM ERC-20 contract

(sha256sum 95848c6e56e7ebc2dcaa48671db12eec8f39ff4de308b1b4f0444989fd6484ee  erc20.abi)

Action 3: add code permission to `eosio.erc2o`

Action 4: Deploy the new deposit proxy contract to the `eosio.evmin` account. It acts as a proxy in the EOS->EVM flow to receive token and redirect received tokens to proper accounts (either eosio.evm for native EOS token, or eosio.erc2o for other tokens). No abi file require for this contract.

(sha256sum 1472893488d7c721f149bede12709eeb75718de397522331866b2b372170b6b5  deposit_proxy.wasm)

Action 5: add code permission to `eosio.evmin`

Action 6: Call bridgereg in EOS EVM Contract for the eosio.erc2o receiver (this also opens the balance for eosio.erc2o). Use 0.01 EOS as minimum bridge fee.

Action 7: Transfer 100 EOS from enf to the eosio.erc2o open balance in the EOS EVM Contract to use as initial bridging funds

Action 8: Call upgradeto action on the erc20 contract to initialize it. (The mainnet implementation contract is 0x9CfbCA2c181425Bd8651AB1587E03c788B081232, and the testnet implementation contract is 0x8ac75488C3B376e13d36CcA6110f985bb65A23c2)

Action 9: Call regtoken on erc20 contract to register the USDT@tethertether token (use JUNGLE@eosio.token for testnet). Use egress fee of 0.01 EOS. User ingress fee of 0.0100 USDT (or 0.0100 JUNGLE on testnet). EVM precision should be 6. Set Name to be same as the symbol. Symbol will be WUSDT on mainnet, WJUNGLE on testnet.

Action 10: Call action to set egress allow list to the same as we have for the EOS EVM Contract.
