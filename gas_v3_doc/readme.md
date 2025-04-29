
<b><h1>Vaulta EVM Gas V3 Design</h1></b>

- **Problem with legacy EVM models:**
The legacy model uses gas price and gas usage for each opcode to define how the final gas fee is calculated. Due to the fact that Antelope blockchain has individual pricing formulas for RAM (storage) cost and CPU (computational) cost, this legacy model is not suitable for a long run.

- **Break transaction's gas usage into 2 parts:**
  - computational gas: the gas consumed for opcodes that do not require additional storage (like math ops, read from storage). We use `overhead price` to reflect how much should be charged per 1 computation gas unit.
  - storage gas: the gas comsumed for opcodes that require storage cost (like newaccount, contract creation, write storage), that would requires additional RAM. We use `storage price` to reflect how much should be charged per 1 storage gas unit.

- **Make things compatible:**
In order to make EVM transaction interface compatible, in such a way that a user will still see only one price and one gas usage, the base gas price will be the max of `overhead price` (computational related) and `storage price`, and the final gas usage will be the sum of computational gas usage and storage gas, discounted by some gas refund.

- **Gas refunds:**
  - If `inclusion price` > 0 in a transaction, the miner will only get the `inclusion price` x `the computional gas` as the miner fee, given the fact that the miner is only required to cover cpu cost but not RAM cost. In this case, if the final gas usage has storage gas, the overcharged portion (`inclusion price` x `storage gas consumption`) will be refunded back to the sender.

  - If `overhead price` > `storage price`, the storage gas usage (if there's any) will be discounted (see example later in this doc).

  - If `overhead price` < `storage price`, the computation gas usage will be discounted.
 

<b><h2>Initial Deployment Steps:</h2></b>

- 1. Set initial overhead price and storage price:
For example, the following command will set overhead price as 5 Gwei and storage price as 2.5 Gwei:
```
price_mb="307.2000 EOS"
overhead_price=5000000000
storage_price=2500000000
./cleos push action eosio.evm setgasprices "{\"prices\":{\"overhead_price\":${overhead_price},\"storage_price\":${storage_price}}}" -p eosio.evm
```

- 2. Upgrade EVM version to 3
```
./cleos push action eosio.evm setversion '[3]' -p eosio.evm
```

- 3. Update gas parameter to reflect the gas consumptions for storage related ops (assume RAM price is 0.3EOS/KB):
```
price_mb="307.2000 EOS"
storage_price=2500000000
./cleos push action eosio.evm updtgasparam "[\"${price_mb}\",\"${storage_price}\"]" -p eosio.evm
```

- 4. Verify the current overhead price, storage price, and gas parameters:
```
./cleos get table eosio.evm eosio.evm config -l 1
{
  "rows": [{
      "version": 0,
      "chainid": 15557,
      "genesis_time": "2025-04-28T05:07:16",
      "ingress_bridge_fee": "0.0100 EOS",
      "gas_price": 0,
      "miner_cut": 0,
      "status": 0,
      "evm_version": {
        "pending_value": null,
        "cached_value": 3
      },
      "consensus_parameter": {
        "cached_value": [
          "consensus_parameter_data_v0",{
            "gas_parameter": {
              "gas_txnewaccount": 40664236,
              "gas_newaccount": 40664236,
              "gas_txcreate": 71015928,
              "gas_codedeposit": 117188,
              "gas_sset": 40549948
            }
          }
        ],
        "pending_value": null
      },
      "token_contract": "eosio.token",
      "queue_front_block": 227,
      "ingress_gas_limit": 21000,
      "gas_prices": {
        "overhead_price": "5000000000",
        "storage_price": 2500000000
      }
    }
  ],
  "more": false,
  "next_key": ""
}
```
In the above example, under the case of RAM price 0.3EOS/KB and storage price 2.5Gwei, the gas consumption of new account ops is 40664236, the gas consumption of contract creation is 7101592. 

You can also verify that for new account ops, it will cost 2.5Gwei x 40664236 = 0.101660590EOS, which translates into around 347 bytes of RAM that covers the RAM cost of storing a new EVM account.


- 5. Try different types of EVM transaction and verify the gas price and gas fee:
    - gas price: at least 5 Gwei. (Max of overhead price & storage price).
    - Basic gas token transfer to existing account: 21000 gas (=0.0001 EOS) (using gas price = 5Gwei)
    - Basic gas token transfer to new account: 20353118 gas (=0.1017 EOS) (It need 40664236 gas to cover the RAM cost in the worst case. Since gas price is 5Gwei, but the storage price is 2.5Gwei, it will discount the gas usage by around 50%)
    - ERC-20 token transfer to existing account: 42620 gas (=0.0002 EOS)
    - ERC-20 token transfer to new account: 20317594 gas (=0.1017 EOS). (similar reason as basic gas token transfer to new account)


<h2><b>Monitor RAM/CPU cost regularly and adjust overhead_price and storage_price to align with the current RAM/CPU cost.</b></h2>

For example, if RAM price has increased from 0.3EOS/KB to 0.6EOS/KB, we then change the storage_price which is 2x of the original using `setgasprices` action. This operation can be done frequently (for example once per 10 minutes) as RAM or CPU price change, but not more frequent than once per 3 minutes.

However, since action `updtgasparam` will re-define the gas costs for storage related operations, which might break the compatibility of existing dapps, this operation should rarely be called unless there is a significant divergence between the current RAM price and the RAM price used in the last `updtgasparam` operation. For example, if the RAM price has been increased from 0.3EOS/KB into 3EOS/KB, or decreased from 0.3EOS/KB to 0.05EOS/KB.

- This is the command example if RAM price has increased to 0.6EOS/KB. 
```
overhead_price=5000000000
storage_price=5000000000
./cleos push action eosio.evm setgasprices "{\"prices\":{\"overhead_price\":${overhead_price},\"storage_price\":${storage_price}}}" -p eosio.evm
```

- wait for 3 minutes until the new prices are effective

- verify the new gas consumption and gas fee (using gas price = 5Gwei)
    - Basic gas token transfer to new account: 40685236 gas (=0.2034 EOS) (Notice that since storage price = gas price = 5Gwei, there is no discount to storage related gas consumption)
    - Basic gas token transfer to existing account: 21000 gas (=0.0001 EOS)

