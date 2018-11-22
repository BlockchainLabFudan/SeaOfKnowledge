# RPC接口



goc的HTTP RPC 大部分与eos 相同，本文将分析[eos 官方资料](https://developers.eos.io/eosio-nodeos/v1.4.0/reference)，形成分析文档。欢迎修改补充。

命令行下的`cleos`  `nodeos`  `keosd` 的执行过程是通过解析参数，再把参数打包，利用HTTP RPC的api，push action 发送到节点。

### get_info

| url  | /v1/chain/get_info |
| -------- | ------ |
| body |   |
|说明 | 链的基本信息 |
|命令 | cleos get info |


```
{
    "server_version": "672aad51",
    "chain_id": "1c6ae7719a2a3b4ecb19584a30ff510ba1b6ded86e1fd8b8fc22f1179c622a32",
    "head_block_num": 561,     //区块高度
    "last_irreversible_block_num": 419, //不可逆区块高度
    "last_irreversible_block_id": "000001a35d70d1a9c1be796ac80d4e3b1e622e37ba58c77bec5b4e7762247a79",
    "head_block_id": "000002311377dbf5b49436a2e7941d6f5dec901d8a9b66877d36671c7ac8b6fe",
    "head_block_time": "2018-11-19T11:44:22.500",
    "head_block_producer": "producer111f",  //生产该区块的bp帐号
    "virtual_block_cpu_limit": 349856,
    "virtual_block_net_limit": 1835861,
    "block_cpu_limit": 99900,
    "block_net_limit": 1048576,
    "server_version_string": "GOC.1011_132-32-g672aad519"
}
```


###  get_block

### get_block_header_state

### get_account

### get_abi

### get_code

### get_raw_code_and_abi

### get_table_rows

具体参考[goc_table]( ./goc_table/goc_table.md )

| url  | /v1/chain/get_table_rows |
| -------- | ------ |
| body | 具体参考[goc_table]( ./goc_table/goc_table.md )  |
|说明 | 表信息 |
|命令 | cleos get table [OPTIONS] account scope table |

### get_currency_balance

###  abi_json_to_bin

### abi_bin_to_json

### get_required_keys

### get_currency_stats

### get_producers

### push_block

### push_transaction

### push_transactions

### get_actions

### get_transaction
