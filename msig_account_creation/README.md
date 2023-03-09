<h1>ENF & EVM account details, multi-sig account creation of eosio.evm</h1>

<h2>Overview:</h2>
For the EVM launch in jungle testnet and later in mainnet, ENF will be creating two main accounts in each environment, respectively. 

- Account "<b>enf</b>": this account will be used to represent on-chain activities that signed by ENF. It will create 4 sub-accounts named admin1.enf, admin2.enf, admin3.enf, admin4.enf, each of those will be associated with a physical person in ENF. It requires 2 out of 4 admin's signatures to satisfy the permission of enf@active.

- Account "<b>eosio.evm</b>": this is the main account for EVM smart contract (<b>evm_runtime.wasm</b> and <b>evm_runtime.abi</b>, to be disclosed later), which has the following main actions:
  - <b>init</b>: action for bootstrapping the EVM chain.
  - <b>pushtx</b>: the main entry point of pushing a signed EVM transaction, including the bridging transaction from EVM to EOS
  - <b>notification on eosio.token::transfer</b>: the notification handler for token bridging from EOS to EVM
  - <b>freeze</b>: emergency action to freeze & unfreeze the EVM contract. If the contract is freezed, users are not allow to push EVM transaction nor use the bridge.
  
<h2>details permission structure of ENF & eosio.evm</h2>

![multisig account creation - architecture](https://user-images.githubusercontent.com/37097018/223971277-225ea5a9-df08-4548-8b62-1dbbdd14110e.png)

As illustrated above:

- Account "enf" will have owner permission satisfied by eosio.grants@active, and active permission will be satisfied by 2 out of 4 admins.

- Account "eosio.evm":
  - owner: this always set to eosio@active, so that 15/21 BP can co-sign to satisfy this
  - active: this permission is mainly for contract deployment / upgrade. As compared to eosio@active which required 15 BP approvals, it is the fast path which allow ENF + 3BPs (out of 5 preselected BPs) to satify. 
    - threshold: 6
    - enf: weight 3
    - pre-seletect BP1: weight 1
    - pre-seletect BP2: weight 1
    - pre-seletect BP3: weight 1
    - pre-seletect BP4: weight 1
    - pre-seletect BP5: weight 1
  - freeze: a special permission to freeze all EVM activities in emergency. this will only linked to the "freeze" action and enf itself can satisfy.
  
  
<h2> multi-sig proposal of creating eosio.evm account </h2>
As account "eosio.evm" started with "eosio.", it requires "eosio@active" permission to create. This requires us to propose a multisig proposal to create the account "eosio.evm".
 
- for testnet, the proposal would be:
  - proposal name: create.evm
  - proposer: any account is fine
  - approver list: the current 21 BPs, plus eosio@active (short cut)
  - proposing transaction: to create eosio.evm with the above permission structure. (with ENF's weight set to 6 to facilitate EVM contract deployment and bootstrapping, ENF will set it back to 3 after bootstrap). 
  - expiring time: around 1 year
    
- in jungle4:
  - to check the current proposed msig transaction: ```cleos -u https://jungle4.cryptolions.io:443 multisig review hello1111111 create.evm```
  - to check the approval status: ```cleos -u https://jungle4.cryptolions.io:443 get table eosio.msig hello1111111 approvals2```
  
  
- for mainnet, the proposal would be:
  - proposal name: create.evm
  - proposer: account that created by enf or related with enf
  - approver list: the current 21 BPs, plus some of the standby BPs.
  - proposing transaction: to create eosio.evm with the above permission structure.
  - expiring time: around 1 year
  
  
