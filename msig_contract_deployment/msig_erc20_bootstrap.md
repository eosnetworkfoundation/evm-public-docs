# Multi-sig ERC20 bridge contract bootstrapping

After the creation of ERC20 bridging accounts (`eosio.erc2o` & `eosio.evmin`), we need to bootstrap the bridging contracts:

## Multi-sig 1: Update the latest version of evm_runtime contract & withdraw funds

Action 1: Update the existing evm_runtime contract, adding supports for the following features required for ERC20 bridging:
  - *call action: this action will allow any account in the EOS side acts as the corresponding account (using deterministric reserverd address) in EVM side*
  - *admincall action: this action allow ENF & BPs to co-sign to satisfy eosio.evm@active and act on behalf of any EVM account in emergency situations.*
  - *event mechanisim: this is the mechanisim allowing event generated in the solidity contract in EVM to be sent as bridge message via inline action in native EOS.*

Action 2: withdraw 300 EOS from the eosio.evm's fee balance to enf for covering RAM and as ERC-20 bridge funds 

## Multi-sig 2: ERC-20 contract bootstrapping

Action 1: Deploy core EVM ERC-20 contract to account `eosio.erc2o`. This contract contains the following actions:
  - *upgrade, upgradeto: set or deploy the ERC-20 upgradable implementation contract into EVM side and register the contract address in eosio.erc2o's multi_index table in EOS side.*
  - *regtoken: deploy the new proxy ERC-20 contract on EVM side (which will call the upgradable implementation contract) and setup the link between existing EOS tokens (such as USDT, IQ, ... etc) and the EVM proxy contract.*
  - *transfer notify handler (EOS->EVM flow implementation): action handler to call the EVM's ERC-20 token mint action (with the memo identifying the destination EVM address) via the eosio.evm contract, upon receiving the correct registered tokens from the EOS side.*
  - *onbridgemsg: event handler (EVM->EOS flow implmenetation): action hanlder to call the EOS inline transfer to transfer the registered token to the destination EOS account upon receiving the EVM event which is genereted from the implementation contract and sent via the eosio.evm runtime contract.*
  - *addregress, removeegress: similar to native EOS bridge, it controls the whilelist of EOS destinaction addresses with deployed code, to prevent misuse of CPU resource.*
  - *setegressfee, setingressfee: actions to control the bridge fee. Ingressfee is charged in form of the native tokens, while egressfee is charged in form on EOS.*
  - *withdrawfee: action to withdraw the accumulated ingress fee*

Action 2: add code permission to `eosio.erc2o`

Action 3: Deploy the new deposit proxy contract to the `eosio.evmin` account. It acts as a proxy in the EOS->EVM flow to receive token and redirect received tokens to proper accounts (either eosio.evm for native EOS token, or eosio.erc2o for other tokens)

Action 4: add code permission to `eosio.evmin`

Action 5: Call bridgereg in EOS EVM Contract for the eosio.erc2o receiver (this also opens the balance for eosio.erc2o). Use 0.01 EOS as minimum bridge fee.

Action 6: Transfer 100 EOS to eosio.erc2o account in the EVM side as initial bridging funds

Action 7: Call upgradeto action on the erc20 contract to initialize it.

Action 8: Call regtoken on erc20 contract to register the USDT@tethertether token (use JUNGLE@eosio.token for testnet). Use egress fee of 0.01 EOS. User ingress fee of 0.0100 USDT (or 0.0100 JUNGLE on testnet). EVM precision should be 6. Set Name to be same as the symbol. Symbol will be WUSDT on mainnet, WJUNGLE on testnet.

Action 7: Call action to set egress allow list to the same as we have for the EOS EVM Contract.
