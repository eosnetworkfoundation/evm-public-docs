<h1>Multi-sig account creation of evm accounts for EOS-ERC20 token bridge </h1>

<h2>Overview</h2>
EOS-EVM will soon support trustless token bridge between tokens on native EOS (for example: USDT on EOS) and ERC-20 compatible tokens on EOS-EVM. (contract implementaion can be found in https://github.com/eosnetworkfoundation/evm-bridge-contracts). Because of such, we need the multi-sig from 21 block producers to create the following accounts on testnet & mainnet:

<h3>account <b>eosio.erc2o</b></h3>

  it is the account that handles the main logic of token bridge operations, such as token registration, token transfer between native EOS and EOS-EVM. The EVM ERC-20 bridge contract will be deployed into this account.

<b>permission structure:</b>
  - owner: set to eosio@active
  - active: set to eosio.evm@active

<b>list of actions:</b>
  - upgrade, upgradeto: set or deploy the ERC-20 upgradable implementation contract into EVM side and register the contract address in eosio.erc2o's multi_index table in EOS side.
  - regtoken: deploy the new proxy ERC-20 contract on EVM side (which will call the upgradable implementation contract) and setup the link between existing EOS tokens (such as USDT, IQ, ... etc) and the EVM proxy contract.
  - transfer notify handler (EOS->EVM flow implementation): action handler to call the EVM's ERC-20 token mint action (with the memo identifying the destination EVM address) via the eosio.evm contract, upon receiving the correct registered tokens from the EOS side.
  - onbridgemsg: event handler (EVM->EOS flow implmenetation): action hanlder to call the EOS inline transfer to transfer the registered token to the destination EOS account upon receiving the EVM event which is genereted from the implementation contract and sent via the eosio.evm runtime contract.
  - addregress, removeegress: similar to native EOS bridge, it controls the whilelist of EOS destinaction addresses with deployed code, to prevent misuse of CPU resource.
  - setegressfee, setingressfee: actions to control the bridge fee. Ingressfee is charged in form of the native tokens, while egressfee is charged in form on EOS.
  - withdrawfee: action to withdraw the accumulated ingress fee

<h3>account <b>eosio.evmin</b></h3>

  it is the account that acts as a proxy in the EOS->EVM flow to receive token and redirect received tokens to proper accounts.

<b>permission structure:</b>
  - owner: set to eosio@active
  - active: set to eosio.evm@active

<b>list of actions:</b>
  - transfer notify handler (EOS-EVM flow): it listens to the transfer action for all the token contracts in EOS side. It validates the amount, memo and other information, and then sends an inline transfer to eosio.evm if receives native EOS tokens, otherwise it sends an inline transfer to eosio.evmin (if it receives other non-native tokens)
 
