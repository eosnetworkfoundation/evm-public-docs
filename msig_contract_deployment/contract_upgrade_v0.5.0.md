<h2>Multi-sig transaction to update the EVM smart contract to version v0.5.0 in EOS mainnet.</h2>

msig link: 
https://bloks.io/msig/admin1.enf/evmupdate

The following actions are included in the above multisig:

Action 1
set code: (hash 9b65b25561e945cbdf23d205b1d6fd9e713b31411ca600c74d814879b15c09ed)

Action 2:
set abi (hash 75a87084267dd1fd295d952a3babb8f9c22b7f4c8ef0c1297fe49ca7bb916737)

The code and abi can be compiled and verfied using the following commit:

eos-evm repo:
https://github.com/eosnetworkfoundation/eos-evm 
commit hash: 12288f15602ff5d65e5e3d6a6a1d459e5d4deead

cdt-cpp version: 3.0.0

new features in this version of evm smart contract:
- Action to execute read-only transactions
- Increased stack size to allow deeper nested calls
- Other improvements to the EOS EVM Contract: reject non-EOS token (such as USDT), etc.

corresponding release notes:
https://github.com/eosnetworkfoundation/eos-evm/releases/tag/v0.5.0-rc2
