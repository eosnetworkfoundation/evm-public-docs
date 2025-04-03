The EVM Trustless Bridge contract provides the trustless bridging solutions between native Antelope blockchain and EVM layer 2 on Antelope blockchain.

**support the following types of tokens:**
- Type 1: Tokens that originated from native Antelope blockchain (such as USDT in native contract tethertether)
- Type 2: ERC-20 compatible tokens that originated from EVM layer 2 on Antelope blockchain

**Advantages:**
- Bridge transfer between native and EVM is performed atomically within a single transaction. Any failure will fails and revert the whole transaction without asset loss.
- Precisions can be different between native side and EVM side. (Usually ERC20 tokens have more precision than native tokens)

**Architectures:**

- type 1: tokens that originated from native
```
Send token from native to EVM:
userabc send 100 USDT to bridgecontract (eosio.erc2o) with memo = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4

userabc          +---- native world ----+              +--- native world --------+ --- EVM world -------+
send native  --> | erc bridge contract  | -- invoke -> | evm_runtime contract    | erc20-token contract | --> issue token to 0x5B38...
token            | (holds native tokens)|              | to make internal evm tx | (deployed by bridge  |
                 |                      |              | to call erc20 contract  | contract at regtoken)|
                 +----------------------+              +-------------------------+ ---------------------+
				 
Send token from EVM back to native:

user 0x5B38...
 |           +---- EVM world -------+                            + native world +            + native world +
 +- call ->  | bridgeTransfer() on  | -- 1. burn token in EVM    |              |            | erc bridge   |
             | erc20-token contract | -- 2. bridgeMessage -----> | evm_runtime  | - notify ->| contract     |-> send token to userabc
             | (deployed by brige)  |                            | contract     |            +--------------+
             +----------------------+                            +--------------+
```


- type 2: tokens that originated from EVM layer 2:

```
Send token from EVM to native:

user 
0x5B38... -- call ERC20-token::approve(sender = portal contract (deployed by the erc bridge contract in regevm2nat action), amount=bridge amount)

user 0x5B38...
 |            +---- EVM world -------+                            + native world +            + native world +
 +-- call ->  | bridgeTransfer() on  | -- 1. get token from user  |              |            | erc bridge   |
              | portal contract      | -- 2. bridgeMessage -----> | evm_runtime  | - notify ->| contract     |-> send mirrored token to userabc
              | (deployed by bridge) |                            | contract     |            +--------------+
              +----------------------+                            +--------------+
					   
Send token from native back to EVM:
userabc send 100 mirrored token to bridgecontract (eosio.erc2o) with memo = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4

userabc          +---- native world ----+              +--- native world --------+ --- EVM world ---------+
send mirrored -> | erc bridge contract  | -- invoke -> | evm_runtime contract    | portal      contract   | --> transfer token to 0x5B38...
token            | (holds native tokens)|              | to make internal evm tx | (deployed by bridge    |
                 |                      |              | to call erc20 contract  | contract at regevm2nat)|
                 +----------------------+              +-------------------------+ -----------------------+
```


**Deployment steps:**

Initial setup:

- Deploy the evm bridge contract (we use account eosio.erc2o as example), set permission, initial funding:
```
cleos set code eosio.erc2o erc20.wasm 
cleos set abi eosio.erc2o erc20.abi
cleos set account permission eosio.erc2o active --add-code
cleos transfer userA eosio.evm "100.0000 EOS" "eosio.erc2o"
```

- Initialize the erc bridge (we use eosio.evm as evm runtime contract account in this example):
```
cleos -u push action eosio.erc2o init "{\"evm_account\":\"eosio.evm\",\"gas_token_symbol\":\"4,EOS\",\"gaslimit\":500000,\"init_gaslimit\":10000000}" -p eosio.erc2o
```

- Call bridgereg action of evm runtime contract:
```
cleos push action eosio.evm bridgereg '["eosio.erc2o","eosio.erc2o","0.0100 EOS"]' -p eosio.erc2o -p eosio.evm@owner
```

- The trustless bridge use the proxy pattern in EVM side. So we need to deploy the implementation contract for Type 1 token.
```
cleos push action eosio.erc2o upgrade '[]' -p eosio.erc2o
```

- Also we need to deploy the implementation contract for Type 2 token.
```
cleos push action eosio.erc2o upgdevm2nat '[]' -p eosio.erc2o
```


Register Trustless Bridge for Type1 Token example (native token USDT):
```
./cleos push action eosio.erc2o regtoken '["tethertether","My USDT Token (EOS)","USDT","0.0100 USDT","0.0100 EOS",6]' -p eosio.erc2o
```

Register Trustless Bridge for Type2 Token example (ERC20 token 0x4d9dbb271ee2962f8becd3b27e3ebcd384ad3171):

create mirrored token (for example GOLD) in native side and issue the mirrored max_supply to eosio.erc2o:
```
./cleos set code goldgoldgold eosio.token.wasm
./cleos set abi goldgoldgold eosio.token.abi
./cleos push action goldgoldgold create '{"issuer":"goldgoldgold", "maximum_supply":"1000000.0000 GOLD", "can_freeze":0, "can_recall":0, "can_whitelist":0}' -p goldgoldgold@active
./cleos push action goldgoldgold issue '{"to":"goldgoldgold", "quantity":"1000000.0000 GOLD", "memo":"issue GOLD"}' -p goldgoldgold@active
```

call regevm2nat action to register type2 bridge:
```
./cleos push action eosio.erc2o regevm2nat '["0x4d9dbb271ee2962f8becd3b27e3ebcd384ad3171","goldgoldgold","0.1000 GOLD","0.0100 EOS",18,""]' -p eosio.erc2o
```

After token registration you need to check the generated ERC-20 token address (for Type1) and the generated portal address (for Type2). This can be done by querying the tables via cleos command:
```
cleos get table eosio.erc2o eosio.erc2o tokens
{
  "rows": [{
      "id": 1,
      "token_contract": "tethertether",
      "address": "33b57dc70014fd7aa6e1ed3080eed2b619632b8e",
      "ingress_fee": "0.0100 USDT",
      "balance": "99.9900 USDT",
      "fee_balance": "0.0100 USDT",
      "erc20_precision": 6
    },{
      "id": 2,
      "token_contract": "goldgoldgold",
      "address": "efb6a2241f3cc0e740a9aab830ef729130667574",
      "ingress_fee": "0.1000 GOLD",
      "balance": "0.0000 GOLD",
      "fee_balance": "0.0000 GOLD",
      "erc20_precision": 18,
      "from_evm_to_native": 1,
      "original_erc20_token_address": "4d9dbb271ee2962f8becd3b27e3ebcd384ad3171"
    }
  ],
  "more": false,
  "next_key": ""
}
```
As in the above example:
For USDT token (type 1), the generated ERC20 token address is 33b57dc70014fd7aa6e1ed3080eed2b619632b8e
For Gold token (type 2), the generated portal address is efb6a2241f3cc0e740a9aab830ef729130667574

Native -> EVM token transfer example:
```
cleos transfer userabc eosio.erc2o "100.0000 USDT" "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4" -c tethertether
```

EVM -> native token transfer example:
```
from https://bridge.evm.eosnetwork.com/, find out the destination reserved address for native account "userabc" is 0xbbbbbbbbbbbbbbbbbbbbbbbbd615731d00000000

for GOLD, we need to approve efb6a2241f3cc0e740a9aab830ef729130667574 as spender:
call 4d9dbb271ee2962f8becd3b27e3ebcd384ad3171::approve(spender = efb6a2241f3cc0e740a9aab830ef729130667574, amount).

for USDT, call 33b57dc70014fd7aa6e1ed3080eed2b619632b8e::bridgeTransfer(address to, uint256 amount, string memory)
or for GOLD, call efb6a2241f3cc0e740a9aab830ef729130667574::bridgeTransfer(address to, uint256 amount, string memory) in remix or any other EVM compatible tools.

with the parameters set to:
- to: 0xbbbbbbbbbbbbbbbbbbbbbbbbd615731d00000000
- amount: 1000000000000000000 (1 USDT/GOLD, assume the token precision is 18)
- memory: empty or any string as "memo" field in native transfer

```
