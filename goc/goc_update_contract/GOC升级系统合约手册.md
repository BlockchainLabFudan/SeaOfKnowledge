# GOC升级系统合约手册

```shell
$ cleos -u http://0.0.0.0:8001 get code -c original_system_contract.wast -a original_system_contract.abi gocio
code hash: df154c2404d12c73a1f18edbc9b037b327f133a8f7bca5f7999ee15b065a3eaf
saving wast to original_system_contract.wast
saving abi to original_system_contract.abi

# 注意修改路径，8001为本次更新leader bp
$ cleos -u http://0.0.0.0:8001 set contract -s -j -d gocio ~/桌面/GOCint/build/contracts/eosio.system | tail -n +2 > upgrade_system_contract_trx.json
Reading WASM from /home/thor/桌面/GOCint/build/contracts/eosio.system/eosio.system.wasm...
Publishing contract...

# leader bp do:
$ mv upgrade_system_contract_trx.json upgrade_system_contract_official_trx.json

# modify the expiration to 50 mins later(<9 hours)


# other bps produce the json and compare it with the leader's one
$ cleos -u http://0.0.0.0:8002 set contract -s -j -d gocio ~/桌面/GOCint/build/contracts/eosio.system | tail -n +2 > upgrade_system_contract_trx2.json
Reading WASM from /home/thor/桌面/GOCint/build/contracts/eosio.system/eosio.system.wasm...
Publishing contract...

$ cleos -u http://0.0.0.0:8003 set contract -s -j -d gocio ~/桌面/GOCint/build/contracts/eosio.system | tail -n +2 > upgrade_system_contract_trx3.json
Reading WASM from /home/thor/桌面/GOCint/build/contracts/eosio.system/eosio.system.wasm...
Publishing contract...

$ diff upgrade_system_contract_official_trx.json upgrade_system_contract_trx2.json
2,4c2,4
<   "expiration": "2018-11-23T15:02:26",
<   "ref_block_num": 21140,
<   "ref_block_prefix": 4196000565,
---
>   "expiration": "2018-11-23T14:18:27",
>   "ref_block_num": 21860,
>   "ref_block_prefix": 3562509249,

$ diff upgrade_system_contract_official_trx.json upgrade_system_contract_trx3.json
2,4c2,4
<   "expiration": "2018-11-23T15:02:26",
<   "ref_block_num": 21140,
<   "ref_block_prefix": 4196000565,
---
>   "expiration": "2018-11-23T14:18:44",
>   "ref_block_num": 21896,
>   "ref_block_prefix": 219514261,


# get chain id
$ cleos -u http://0.0.0.0:8003 get info
"chain_id": "1c6ae7719a2a3b4ecb19584a30ff510ba1b6ded86e1fd8b8fc22f1179c622a32"

# start sign
$ cleos -u http://0.0.0.0:8001 sign --chain-id 1c6ae7719a2a3b4ecb19584a30ff510ba1b6ded86e1fd8b8fc22f1179c622a32 upgrade_system_contract_official_trx.json | tail -n 5
"/usr/local/eosio/bin/keosd" launched
private key:   "signatures": [
    "SIG_K1_KcwinQ4i4VuMiB1YNgNgChKk8JzpcR58CUBEKqdseQvJKzqYy5n41aqpYkLTYXRzHqcb7jjA6btgNjtPEqYKEqxAc9pdKP"
  ],
  "context_free_data": []
}

$ cleos -u http://0.0.0.0:8002 sign --chain-id 1c6ae7719a2a3b4ecb19584a30ff510ba1b6ded86e1fd8b8fc22f1179c622a32 upgrade_system_contract_official_trx.json | tail -n 5
"/usr/local/eosio/bin/keosd" launched
private key:   "signatures": [
    "SIG_K1_KfvByGAAEwKaZwQu3gxejbw4qJ8WPouP4NE5BnqpHXd4oTFJxTTJ9LVvSYrqYitryUH15ukskJbHYXKUYayPdSeN4qfKQy"
  ],
  "context_free_data": []
}

$ cleos -u http://0.0.0.0:8003 sign --chain-id 1c6ae7719a2a3b4ecb19584a30ff510ba1b6ded86e1fd8b8fc22f1179c622a32 upgrade_system_contract_official_trx.json | tail -n 5
"/usr/local/eosio/bin/keosd" launched
private key:   "signatures": [
    "SIG_K1_KZczAzeMTdf3Kycj8r3ujTueeTuEKk5UUkbAiH3J5SyQSjPydepi4fTWkSPAtYA8zoLf38WK7wj6KtPfiVDzwR7v87JSjz"
  ],
  "context_free_data": []
}


# leader bp do:
$ cp upgrade_system_contract_official_trx.json upgrade_system_contract_official_trx_signed.json

# modify upgrade_system_contract_official_trx_signed.json to add signatures
$ cat upgrade_system_contract_official_trx_signed.json | tail -n 7
"transaction_extensions": [],
  "signatures": ["SIG_K1_KcwinQ4i4VuMiB1YNgNgChKk8JzpcR58CUBEKqdseQvJKzqYy5n41aqpYkLTYXRzHqcb7jjA6btgNjtPEqYKEqxAc9pdKP",
		 "SIG_K1_KfvByGAAEwKaZwQu3gxejbw4qJ8WPouP4NE5BnqpHXd4oTFJxTTJ9LVvSYrqYitryUH15ukskJbHYXKUYayPdSeN4qfKQy",
		 "SIG_K1_KZczAzeMTdf3Kycj8r3ujTueeTuEKk5UUkbAiH3J5SyQSjPydepi4fTWkSPAtYA8zoLf38WK7wj6KtPfiVDzwR7v87JSjz"
],
  "context_free_data": []
}

$ cleos -u http://0.0.0.0:8001 push transaction --skip-sign upgrade_system_contract_official_trx_signed.json

$ cleos -u http://0.0.0.0:8002 get code -c new_system_contract.wast -a new_system_contract.abi gocio
code hash: 71b32cf1bedf20eaef2bf7d0fea118546d17d639ae3da6d4cb58c1424f618812
saving wast to new_system_contract.wast
saving abi to new_system_contract.abi

# 本次合约更新是将gocstake数值从1000GOC更改到100000GOC
$ diff original_system_contract.wast new_system_contract.wast
59765c59765
<     i64.const 10000000
---
>     i64.const 1000000000
59937c59937
<           i64.const 10000000
---
>           i64.const 1000000000
60638c60638
<       i64.const 10000000
---
>       i64.const 1000000000
62016c62016
<     i64.const 10000000
---
>     i64.const 1000000000
62798c62798
<     i64.const 10000000
---
>     i64.const 1000000000
```

