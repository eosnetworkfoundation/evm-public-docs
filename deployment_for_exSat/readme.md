# Deploy and Support exSat EVM Network for exSat Node operators

This document will describes the minimum requirements to deploy and support exSat EVM Network.

<!-- contents box begin -->
<table>
<tr/>
<tr>
<td>
<p/>
<div align="center">
<b>Contents</b>
</div>
<p/>
<!-- contents markdown begin -->

1. [Minimum Architecture](#MA)<br/>
2. [Building necessary components](#BNC)<br/>
3. [Running the native chain node](#REN)<br/>
4. [Running the exSat EVM chain node and rpc service](#REE)<br/>
5. [Running the exSat EVM miner service](#RMS)<br/>
6. [[Optional]: Setting up the read-write proxy](#RWP)<br/>
6a. [[Optional]: An alternative way to the read-write proxy](#RWP2)<br/>
7. [Backup & Recovery of native node & exSat EVM node](#BR)<br/>
8. [[Emergency Only]: Replay the EVM chain for major version upgrades](#REPLAY)<br/>
9. [Known Limitations](#KL)<br/>

<!-- contents markdown end -->
<p/>
</td>
</tr>
</table>
<!-- contents box end -->

<a name="MA"></a>
## Minimum Architecture 
This is the minimum setup to run an exSat EVM service. It does not contain the high availability setup. Exchanges can duplicate the real-time service part for high availability purpose if necessary. 
```
Real-time service:
                                      +------ exSat EVM Chain VM -----+
+------ native chain VM -------+      | eos-evm-node (main process of |
| spring node (main process of | <--- | exSat EVM chain), eos-evm-rpc | <-- eth compatible read requests (e.g. eth_getBlockByNumber, eth_call ...)
| the native chain)            |      +-------------------------------+
+------------------------------+                                    \      + ----- exSat EVM Chain VM---------------------------+
                ^                                                    <---- | proxy to separate read & write requests (optional) |
                |                                                   /      + ---------------------------------------------------+
                |                                                  /
                |                              +--- exSat EVM Chain VM ----+
                \-- push native transactions --| eos-evm-miner (tx wrapper)| <-- eth_gasPrice / eth_sendRawTransaction
                                               +---------------------------+												        
```

<b>spring node</b>: stands for the native (Level 1) blockchain.<br/>
<b>eos-evm-node</b>: the main process for the exSat EVM (Level 2) blockchain.<br/>
<b>eos-evm-rpc</b>: a separate process that talks to eos-evm-node in the same VM, providing read-only ETH APIs (such as eth_getBlockByNumber, eth_call, eth_blockNumber, ... ) that are compatible with standard ETH API. <br/>
<b>eos-evm-miner</b>: process that handles eth_sendRawTransaction and eth_gasPrice, wrapping ETH transactions into native transactions.<br/>
<b>proxy</b>: [Optional] Proxy to separate read requests and write requests, providing the same ETH compatible endpoint to clients.

Hardware requirements:

- native chain VM:    RAM minimum 32GB (64GB+ recommended). SSD 750GB+ (may grow as native chain data grows). 
- exSat EVM Chain VM: RAM minimum 16GB (32GB+ recommended). SSD 250GB+ (may grow as exSat chain data grows)


<a name="BNC"></a>
## Building necessary components:
OS: Recommend to use ubuntu 22.04

- Native (spring) Node: the main process for native chain
please refer to https://github.com/AntelopeIO/spring
The latest pre-built stable version can be downloaded from https://github.com/AntelopeIO/spring/releases

- eos-evm-node, eos-evm-rpc: the main process for exSat EVM chain
Please build the latest stable version in https://github.com/eosnetworkfoundation/eos-evm-node

for example, the latest stable version is v1.0.1 up to the time of writing this doc.
```
git clone https://github.com/eosnetworkfoundation/eos-evm-node.git
cd eos-evm-node
git checkout v1.0.1
git submodule update --init --recursive
mkdir build; cd build;
cmake .. && make -j8
```

- eos-evm-miner: please refer to https://github.com/eosnetworkfoundation/eos-evm-miner

- Proxy to separate read requests & write requests: please refer to https://github.com/eosnetworkfoundation/eos-evm-node/tree/main/peripherals/proxy

<a name="REN"></a>
## Running the native chain node

### create a 256GB swap and 240GB tmpfs system to hold the native blockchain state

example script:
```
#!/bin/bash

size=256G
tmpfssize=240G

r=`df -h | grep /mnt/d | grep $tmpfssize | wc -l`

if [[ r -eq 1 ]]; then
  echo "tmpfs /mnt/d already setup with the expected size of $tmpfssize"
  exit 0
fi

echo "=== umount /mnt/d ==="
sudo umount /mnt/d

echo "=== swapoff and remove existing swapfile ==="
sudo swapoff -a
sudo rm /swapfile

echo "=== make a swap with size $size ==="

sudo fallocate -l $size /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo swapon --show

r=`cat /etc/fstab | grep /swapfile | wc -l`

if [[ r -eq 0 ]]; then
  echo "adding /swapfile into /etc/fstab"
  sudo cp /etc/fstab /etc/fstab.bak
  echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
else
  echo "/swapfile already in /etc/fstab"
fi

echo "==== mount tmpfs of size $tmpfssize to /mnt/d ==="
sudo mkdir -p /mnt/d
sudo mount -t tmpfs -o size=$tmpfssize tmpfs /mnt/d
sudo chmod a+rwx /mnt/d
```

### download a proper snapshot

- For the first time: You need a snapshot file whose timestamp is before the exsat EVM genesis timestamp 2024-10-09T07:54:41 UTC. You can download the snapshot from any public antelope snapshot service providers (such as https://snapshots.eosnation.io/), or use your own snapshot.

- You need to keep the block logs, state-history logs starting from the snapshot point. This is because eos-evm-node may ask for old blocks for replaying the EVM chain. 

### prepare the config

example data-dir/config.ini for native chain node

```
# 180GB chain-base size, using swap & tmpfs
chain-state-db-size-mb = 184320
access-control-allow-credentials = false

allowed-connection = any
p2p-listen-endpoint = 0.0.0.0:9876
p2p-max-nodes-per-host = 10
http-server-address = 0.0.0.0:8888
state-history-endpoint = 0.0.0.0:8999

trace-history = true
chain-state-history = false
http-max-response-time-ms = 1000

# add/remove p2p peers if necessary
p2p-peer-address=eos.p2p.eosusa.io:9882
p2p-peer-address=p2p.eos.cryptolions.io:9876
p2p-peer-address=p2p.eossweden.se:9876

max-transaction-time = 499
read-only-read-window-time-us = 1000000
transaction-retry-max-storage-size-gb = 1

# Plugin(s) to enable, may be specified multiple times
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
plugin = eosio::producer_api_plugin
plugin = eosio::state_history_plugin
plugin = eosio::net_plugin
plugin = eosio::net_api_plugin
plugin = eosio::db_size_api_plugin
```

### Run the native node process

For the 1st time, we need to run the spring node with an initial snapshot:
```
./nodeos --p2p-accept-transactions=0 --state-dir=/mnt/d --data-dir=./data-dir --config-dir=./data-dir --http-max-response-time-ms=1000 --disable-replay-opts --max-body-size=10000000 --snapshot=SNAPSHOT_FILE
```

After that, any restart of the spring node will not need the snapshot argument:
```
./nodeos --p2p-accept-transactions=0 --state-dir=/mnt/d --data-dir=./data-dir --config-dir=./data-dir --http-max-response-time-ms=1000 --disable-replay-opts --max-body-size=10000000
```

Notes:
- To boost performance, it is important to set "--p2p-accept-transactions=0" to disallow executing transactions (which are not yet included in a blocks) received from other peers.


### Verify the native chain node.
use `./cleos get info` to check if the node the in-sync with the native network (via the head_block_time), for example:
```
./cleos get info
{
  "server_version": "57465074",
  "chain_id": "aca376f206b8fc25a6ed44dbdc66547c36c6c33e3a119ffbeaef943642f0e906",
  "head_block_num": 399277762,
  "last_irreversible_block_num": 399277760,
  "last_irreversible_block_id": "17cc7ec09ef0f9d50c7a7da3751f370ae215912d1f4588b968f4e1d14a257858",
  "head_block_id": "17cc7ec2498e4dae12046843502ec8c88bfdbee801b96aa1ba58ea6e3f2508d6",
  "head_block_time": "2024-10-14T02:30:34.000",
  "head_block_producer": "big.one",
  "virtual_block_cpu_limit": 200000,
  "virtual_block_net_limit": 1048576000,
  "block_cpu_limit": 200000,
  "block_net_limit": 1048576,
  "server_version_string": "v1.0.1",
  "fork_db_head_block_num": 399277762,
  "fork_db_head_block_id": "17cc7ec2498e4dae12046843502ec8c88bfdbee801b96aa1ba58ea6e3f2508d6",
  "server_full_version_string": "v1.0.1-574650744460373f635d48cac9aa6dee67dcbfdb",
  "total_cpu_weight": "382730335491165",
  "total_net_weight": "95742723072332",
  "earliest_available_block_num": 399104422,
  "last_irreversible_block_time": "2024-10-14T02:30:33.000"
}
```

### verify the chain-state (database) size usage: 
use `curl http://127.0.0.1:8888/v1/db_size/get`, for example
```
curl http://127.0.0.1:8888/v1/db_size/get 2>/dev/null | jq
{
  "free_bytes": "92051578720",
  "used_bytes": "101221948576",
  "reclaimable_bytes": 157837488,
  "size": "193273527296",
```


<a name="REE"></a>
## Running the exSat EVM chain node and rpc service

- Make the genesis.json file for exSat EVM chain as follows:
`
{ "alloc": { "0x0000000000000000000000000000000000000000": { "balance": "0x0000000000000000000000000000000000000000000000000000000000000000" } }, "coinbase": "0x0000000000000000000000000000000000000000", "config": { "chainId": 7200, "homesteadBlock": 0, "eip150Block": 0, "eip155Block": 0, "byzantiumBlock": 0, "constantinopleBlock": 0, "petersburgBlock": 0, "istanbulBlock": 0, "trust": {} }, "difficulty": "0x01", "extraData": "exSatEVM", "gasLimit": "0x7ffffffffff", "mixHash": "0x17bfe8ef72abc24e9729bd3037843c7ce1c8022eae521e20e6a744964d1be111", "nonce": "0x56e40ee0d9000000", "timestamp": "0x670636c1" }
`

- run the exSat EVM chain node
```
mkdir ./chain-data
./eos-evm-node --ship-endpoint=<Native chain state-history-endpoint IP:PORT> --ship-core-account evm.xsat --chain-data ./chain-data --plugin block_conversion_plugin --plugin blockchain_plugin --nocolor 1  --verbosity=4 --genesis-json=./genesis.json
```
you should able to see the outputs similar as following:
```
  INFO [10-16|09:49:52.745 UTC] Created DB environment at location : ./chain-data/chaindata
  INFO [10-16|09:49:52.745 UTC] Created Ethereum Backend with network id <7'200>
  INFO [10-16|09:49:52.745 UTC] BackEndKvServer created listening on: 127.0.0.1:8080
  INFO [10-16|09:49:52.745 UTC] Initializing Engine Plugin
  INFO [10-16|09:49:52.745 UTC] Initialized SHiP Receiver Plugin
  INFO [10-16|09:49:52.745 UTC] Determining effective canonical header.
  INFO [10-16|09:49:52.745 UTC] Stored LIB at: #10'000
  INFO [10-16|09:49:52.745 UTC] Search for block containing a valid eos id. Start from:#10'000
  INFO [10-16|09:49:52.745 UTC] load_head: #10'000, txs:0, hash:2ecef7788e3bbf8099d559c2a6cf9069bdb21c0c6d106a2817700096392c3a65
  INFO [10-16|09:49:52.745 UTC] Loaded native block: [10'000][398'472'974],[1'728'470'481'000'000]
  INFO [10-16|09:49:52.745 UTC] Block interval (in seconds): 1
  INFO [10-16|09:49:52.745 UTC] Genesis timestamp (in seconds since Unix epoch): 1'728'460'481
  INFO [10-16|09:49:52.745 UTC] Genesis nonce (as hex): 0x56e40ee0d9000000
  INFO [10-16|09:49:52.745 UTC] Genesis nonce (as Antelope name): evm.xsat
  INFO [10-16|09:49:52.745 UTC] Initialized block_conversion Plugin
  INFO [10-16|09:49:52.745 UTC] Using DB environment at location : ./chain-data/chaindata
  INFO [10-16|09:49:52.745 UTC] Initialized Blockchain Plugin
  INFO [10-16|09:49:52.747 UTC] Started Engine Server
  INFO [10-16|09:49:52.747 UTC] Started SHiP Receiver
  INFO [10-16|09:49:52.748 UTC] Starting Blockchain Plugin
  INFO [10-16|09:49:52.753 UTC] Connected to SHiP at 127.0.0.1:7070
  INFO [10-16|09:49:52.753 UTC] Start Syncing blocks.
  INFO [10-16|09:49:52.753 UTC] Determining effective canonical header.
  INFO [10-16|09:49:52.753 UTC] Stored LIB at: #10'000
  INFO [10-16|09:49:52.753 UTC] Search for block containing a valid eos id. Start from:#10'000
  INFO [10-16|09:49:52.753 UTC] Get_head_canonical_header: #10'000, hash:2ecef7788e3bbf8099d559c2a6cf9069bdb21c0c6d106a2817700096392c3a65, mixHash:17c0370e54ec3b96eaaad7fe0783879a6174b887aef50ea17e37ef8a7402fbc5
  INFO [10-16|09:49:52.753 UTC] Canonical header start from block: 398'472'975
  INFO [10-16|09:49:52.753 UTC] Starting from block #398'472'975
  INFO [10-16|09:49:52.753 UTC] Storing EVM Lib: #10'000
  INFO [10-16|09:49:52.753 UTC] ExecutionEngine                    verifying chain 2ecef7788e3bbf8099d559c2a6cf9069bdb21c0c6d106a2817700096392c3a65
  INFO [10-16|09:49:52.754 UTC] ExecPipeline                       Forward start --------------------------
  INFO [10-16|09:49:52.754 UTC] [1/11 Headers]                     End, forward skipped, we are already at the target block (10'000)
  INFO [10-16|09:49:52.754 UTC] ExecPipeline                       Forward done ---------------------------
  INFO [10-16|09:49:55.642 UTC] Storing EVM Lib: #15'000
  INFO [10-16|09:49:55.642 UTC] ExecutionEngine                    verifying chain 3d863b50f2d2bea79fe2682b2f85dae2ff966f25ca9384f224f27f7d71ae49cf
  INFO [10-16|09:49:55.646 UTC] ExecPipeline                       Forward start --------------------------
  INFO [10-16|09:49:55.646 UTC] [1/11 Headers]                     Updating headers from=10'000
  INFO [10-16|09:49:55.656 UTC] [1/11 Headers]                     Updating completed, wrote 5'000 headers, last=15'000
  INFO [10-16|09:49:55.742 UTC] [1/11 Headers]                     op=Forward done=95.233ms
  INFO [10-16|09:49:55.742 UTC] [2/11 BlockHashes]                 op=Forward from=10000 to=15000 span=5000
  INFO [10-16|09:49:55.754 UTC] [2/11 BlockHashes]                 op=Forward done=12.203ms
  INFO [10-16|09:49:55.777 UTC] [3/11 Bodies]                      op=Forward done=23.288ms
  INFO [10-16|09:49:55.777 UTC] [4/11 Senders]                     op=parallel_recover num_threads=20 max_batch_size=159783
  INFO [10-16|09:49:55.797 UTC] [4/11 Senders]                     op=Forward done=19.992ms
  INFO [10-16|09:49:55.797 UTC] [5/11 Execution]                   op=Forward from=10001 to=15000 span=5000
  INFO [10-16|09:49:55.802 UTC] Flushed history                    size=43.95 KB in=187us
  INFO [10-16|09:49:55.802 UTC] Flushed state                      size=0.00 B in=1us
  INFO [10-16|09:49:55.824 UTC] [5/11 Execution] commit            batch time=21.165ms
  INFO [10-16|09:49:55.824 UTC] [5/11 Execution]                   op=Forward done=26.494ms
  INFO [10-16|09:49:55.824 UTC] [6/11 HashState]                   op=Forward from=10000 to=15000 span=5000
  INFO [10-16|09:49:55.835 UTC] [6/11 HashState]                   op=Forward done=11.377ms
  INFO [10-16|09:49:55.835 UTC] [7/11 IntermediateHashes] begin    op=Forward from=10000 to=15000 span=5000
  INFO [10-16|09:49:55.853 UTC] [7/11 IntermediateHashes]          op=Forward done=17.749ms
  INFO [10-16|09:49:55.853 UTC] [8/11 HistoryIndex]                op=Forward from=10000 to=15000 span=5000
  INFO [10-16|09:49:55.889 UTC] [8/11 HistoryIndex]                op=Forward done=36.361ms
  INFO [10-16|09:49:55.889 UTC] [9/11 LogIndex]                    op=Forward from=10000 to=15000 span=5000
  INFO [10-16|09:49:55.898 UTC] [10/11 TxLookup]                   op=Forward from=10000 to=15000 span=5000
  INFO [10-16|09:49:55.916 UTC] [10/11 TxLookup]                   op=Forward done=18.166ms
  INFO [10-16|09:49:55.936 UTC] [11/11 Finish]                     op=Forward done=19.863ms
  INFO [10-16|09:49:55.936 UTC] ExecPipeline                       Forward done ---------------------------
  INFO [10-16|09:49:59.865 UTC] Storing EVM Lib: #20'000
```


- run the exSat EVM rpc services (must be in the same VM of exSat EVM chain node)
```
./eos-evm-rpc --api-spec=eth,debug,net,trace --chain-id=7200 --http-port=0.0.0.0:8881 --eos-evm-node=127.0.0.1:8080 --chaindata=./chain-data
```
you should able to see outputs similar as following:
```
       [10-16|09:52:34.122 UTC] Silkrpc build info: eos-evm-rpc version: v1.0.1-d10d26a9305fb3870446098349d8be517a853abd
       [10-16|09:52:34.122 UTC] Silkrpc libmdbx version: v0.12.0-71-g1cac6536 build: x86_64-ELF-Linux-Release compiler: cc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
       [10-16|09:52:34.122 UTC] Silkrpc launched with datadir "./chain-data" using 10 contexts, 16 workers
       [10-16|09:52:34.155 UTC] Skip protocol version compatibility check with core services
       [10-16|09:52:34.155 UTC] Starting ETH RPC API at 0.0.0.0:8881 ENGINE RPC API at
       [10-16|09:52:34.157 UTC] Silkrpc is now running [pid=1'071'147, main thread=137'373'153'232'448]
```


- The EVM state, logs will be stored in ./chain-data directory

The eos-evm-rpc will talk to eos-evm-node and provide the eth compatible RPC services, for example, you can check the current block number of eos-evm-node via:
```
curl --location --request POST '127.0.0.1:8881/' --header 'Content-Type: application/json' --data-raw '{"method":"eth_blockNumber","params":["0x1",false],"id":0}'
```
example output:
```
{"id":0,"jsonrpc":"2.0","result":"0xa4e03"}
```


<a name="RMS"></a>
## Running the exSat EVM miner service 
The miner service will help to package the EVM transaction into native transaction and set to the native network. It will provide the following 2 eth API:
- eth_gasPrice: retrieve the currect gas price from native Network
- eth_sendRawTransaction: package the exSat ETH transaction into native transaction and push into the native Network.
clone the https://github.com/eosnetworkfoundation/eos-evm-miner repo

- create your miner account (in this document we use account `a123` as an example) on the native network
- open account balance on EVM side:
  ```
  ./cleos push action evm.xsat open '{"owner":"a123"}' -p a123
  ```
- powerup the miner account with enough CPU & NET resource (for example: 1min CPU. 10 MB net per day). You can use some existing auto powerup service such as https://eospowerup.io/auto or push the powerup transaction (eosio::powerup) via cleos.

- prepare the .env file with the correct information (replace with your own miner account & private_key, set RPC_ENDPOINTS to native chain VM IP)
```
PRIVATE_KEY=5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
MINER_ACCOUNT=a123
EVM_ACCOUNT=evm.xsat
RPC_ENDPOINTS=http://127.0.0.1:8888|http://192.168.1.1:8888|https://eos.greymass.com
PORT=18888
LOCK_GAS_PRICE=false
MINER_PERMISSION=active
EXPIRE_SEC=300
```
- build and start the miner
```
npm install
yarn build
yarn start
```
- test if the miner is working
```
curl http://127.0.0.1:18888 -X POST -H "Accept: application/json" -H "Content-Type: application/json" --data '{"method":"eth_gasPrice","params":[],"id":1,"jsonrpc":"2.0"}'
{"jsonrpc":"2.0","id":1,"result":"0x22ecb25c00"}
```

<a name="RWP"></a>
## [Optional] Setting up the read-write proxy

Follow the build process in https://github.com/eosnetworkfoundation/eos-evm-node/tree/main/peripherals/proxy

The proxy program will separate Ethereum's write requests (such as eth_sendRawTransaction,eth_gasPrice) from other requests (treated as read requests). The write requests should go to Transaction Wrapper (which wrap the ETH transaction into Antelope transaction and sign it and push to the Antelope blockchain). The read requests should go to eos-evm-rpc.

In order to get it working, docker is required. To install docker in Linux, see https://docs.docker.com/engine/install/ubuntu/

```shell
cd eos-evm-node/peripherals/proxy/
```

- Edit the file `nginx.conf`, find the follow settings:

```json
  upstream write {
    server 192.168.56.101:18888;
  }
  
  upstream read {
    server 192.168.56.101:8881;
  }
```

- Change the IP and port of the write session to your Transaction Wrapper server endpoint.
- Change the IP and port of the read session to your eos-evm-rpc server endpoint
- Build the docker image for the proxy program:

```shell
sudo docker build .
```

- Check the image ID after building the image

```shell
sudo docker image ls
```

Example output:

```txt
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
<none>       <none>    49564d312df7   2 hours ago     393MB
debian       jessie    3aaeab7a4777   19 months ago   129MB
```

- Run the proxy in docker:

```shell
sudo docker run -p 81:80 -v ${PWD}/nginx.conf:/etc/nginx.conf 49564d312df7
```

In the commmand above we map the host port 81 to the port 80 inside the docker.

- Check if the proxy is responding:

```shell
echo "=== test 8881 rpc ==="
curl --location --request POST '127.0.0.1:8881/' --header 'Content-Type: application/json' --data-raw '{"method":"eth_blockNumber","params":["0x1",false],"id":0}'
echo "=== test 80 proxy->rpc ==="
curl --location --request POST '127.0.0.1:80/' --header 'Content-Type: application/json' --data-raw '{"method":"eth_blockNumber","params":["0x1",false],"id":0,"jsonrpc":"2.0"}'
echo "=== test 18888 wrapper ==="
curl http://127.0.0.1:18888 -X POST -H "Accept: application/json" -H "Content-Type: application/json" --data '{"method":"eth_gasPrice","params":[],"id":1,"jsonrpc":"2.0"}'
echo ""
echo "=== test 80 proxy->wrapper ==="
curl http://127.0.0.1:80 -X POST -H "Accept: application/json" -H "Content-Type: application/json" --data '{"method":"eth_gasPrice","params":[],"id":1,"jsonrpc":"2.0"}'
echo ""
```

Example response:
```
=== test 8881 rpc ===
{"id":0,"jsonrpc":"2.0","result":"0x2f0d68f"}
=== test 80 proxy->rpc ===
{"id":0,"jsonrpc":"2.0","result":"0x2f0d68f"}
=== test 18888 wrapper ===
{"jsonrpc":"2.0","id":1,"result":"0x45d964b800"}
=== test 80 proxy->wrapper ===
{"jsonrpc":"2.0","id":1,"result":"0x45d964b800"}
```

You can now use endpoint `http://127.0.0.1:80` in metamask for your own exSat EVM account operations.


<a name="RWP2"></a>
## [Optional] An alternative way to setting up the read write proxy:
The following python program provides a simple way for the proxy (change the READ_RPC_ENDPOINT, WRITE_RPC_ENDPOINT and SERVER_PORT if necessary):

```
#!/usr/bin/env python3
import random
import os
import json
import time
import calendar
from datetime import datetime

from flask import Flask, request, jsonify
from flask_cors import CORS
from eth_hash.auto import keccak
import requests
import json

from binascii import unhexlify

readEndpoint = os.getenv("READ_RPC_ENDPOINT","http://127.0.0.1:8881")
writeEndpoint = os.getenv("WRITE_RPC_ENDPOINT", "http://127.0.0.1:18888")
listenPort = os.getenv("SERVER_PORT", 5000)
ssl_cert = os.getenv("SSL_CERT","")
ssl_keyfile = os.getenv("SSL_KEYFILE","")

writemethods = {"eth_sendRawTransaction","eth_gasPrice"}

try:
    app = Flask(__name__)
    CORS(app)

    @app.route("/", methods=["POST"])
    def default():
        def forward_request(req):
            if type(req) == dict and ("method" in req) and (req["method"] in writemethods):
                print("write req:" + str(req))
                resp = requests.post(writeEndpoint, json.dumps(req), headers={"Accept":"application/json","Content-Type":"application/json"}).json()
                print("write resp:" + str(resp))
                return resp
            else:
                resp = requests.post(readEndpoint, json.dumps(req), headers={"Accept":"application/json","Content-Type":"application/json"}).json()
                print("resp is:" + str(resp))
                return resp;

        request_data = request.get_json()
        if type(request_data) == dict:
            print("req is:" + str(request_data));
            return jsonify(forward_request(request_data))

        res = []
        for r in request_data:
            res.append(forward_request(r))

        return jsonify(res)

    if len(ssl_cert) > 0 and len(ssl_keyfile) > 0:
        print("Running in SSL mode")
        app.run(host='0.0.0.0',port=listenPort,ssl_context=(ssl_cert, ssl_keyfile))
    else:
        print("Running in non-SSL mode")
        app.run(host='0.0.0.0', port=listenPort)
finally:
    exit(0)
```

<a name="BR"></a>
## Backup & Recovery of native node & exSat EVM node
The Backup & Recovery is not a must at the beginning, as you can always setup everything from scratch. However, in the middle to long term, 
it is quite important for node operator to backup all the state periodically.

A backup must be done on the spring node running in irreversible mode. And because of such, all the blocks in eos-evm-node has been finialized and it will never has a fork.

```
Periodic Backup service (not mandatory at the beginning, but highly recommended to have) : 
    +----------------- Backup VM ----------------+        +-------- Back up VM -----------+
    | spring node (running in irreversible mode) | <----- | eos-evm-node & eos-evm-rpc    | 
    +--------------------------------------------+        +-------------------------------+         
```
<b>Backup VM</b>: RAM minimum 32GB (64GB+ recommended). SSD 1TB+ (may grow as either native or exSat chain data grows)<br/>
<b>spring node for backup config</b>: add `read-mode = irreversible` to backup spring node to run as irreversible mode.

<b>Backup steps</b>:
- create the nodeos (spring) snapshot:
  ```
  curl http://127.0.0.1:8888/v1/producer/create_snapshot
  ```
- gracefull kill all processes:
```
pkill eos-evm-node
sleep 2.0
pkill eos-evm-rpc
sleep 2.0
pkill nodeos
```
- backup spring's data-dir folder and eos-evm-node's chain-data
- restart back nodeos, eos-evm-node & eos-evm-rpc
- for spring recovery, please restore the data-dir folder of the last successful backup and use the last successful spring's snapshot to start with.
- for eos-evm-node recovery, please restore the chain-data folder of the last successful backup.


<a name="REPLAY"></a>
## [Emergency only] Replay the EVM chain for major version upgrades
Sometime full EVM chain is required if there's a major version upgrade of eos-evm-node. This is the suggested replay process:
- 1. Use the backup VM (in which spring node is running in irreversible mode so that it won't be any forks) for replaying
- 2. Gracefully shutdown eos-evm-rpc & eos-evm-node, keep spring node running.
- 3. Backup the eos-evm-node data folder (specified in --chain-data parameter).
- 4. Delete everything in the --chain-data folder, but keep the folder itself
- 5. Replace the eos-evm-node & eos-evm-rpc to the new version
- 6. Start eos-evm-node & eos-evm-rpc again. Replay will be started automatically.
- 7. Query the current replay process, normally replay will finished within hours:
```
curl --location --request POST '127.0.0.1:8881/' --header 'Content-Type: application/json' --data-raw '{"method":"eth_blockNumber","params":["0x1",false],"id":0}'
```
- 8. After replay finishes, gracefully shutdown eos-evm-rpc & eos-evm-node. Make the evm backup of data-dir folder
- 9. Apply the new binaries & the backup to other eos-evm node.


<a name="KL"></a>
## Known Limitations
- Eos-evm-node will gracefully stop if the state-history-plugin connection in Spring node is dropped. Node operators need to have auto-restart script to restart eos-evm-node (and choose the available spring end-point if high availability setup exist)

- In some rare case, eos-evm-node can not handle forks happened in native chain. Node operators may need to run the recovery process.

- If eos-evm-node crashes, in some case it may not able to start due to database error. Node operators may need to run the recovery process.
