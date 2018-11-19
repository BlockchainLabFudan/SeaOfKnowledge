# GOC table 参考

下表中的信息可以通过RPC api `/v1/chain/get_table_rows` 获取，body示例

```json
{
	"code":"gocio",
	"scope": "gocio",
	"table":"rammarket",
	"json":true,
	"limit":1
}
```

也可以通过命令`./cleos get table [OPTIONS] account scope table`获取。

`返回值` 是指返回的信息
`生成条件` 是指满足该条件会生成该表
`删除条件` 是指满足该条件会删除该表


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     |gocio   |proposals | 所有的提案信息 | 创建提案 | 不会删除 |
|gocio     |int64[id]      |votes     | 对该提案的进行了投票的帐号的信息 | 提案存在，至少有一个帐号投票 |  |
|gocio     |int64[id]      |bpvotes     | 对该提案的进行了投票的BP节点信息 |  |  |
|gocio     |string[name]   |gocrewards |该帐号对的投票信息以及因为投票而得到的奖励信息   |  |  |
|gocio |string[name] |gocvrewards |该帐号投票BP节点得到的奖励信息 |  |  |
|gocio     | string[name] |userres | 该帐号基本信息 |  |  |
|gocio     | gocio |global | 链的全局状态信息 |  |  |
| gocio | gocio | producers | 注册为生产者的帐号信息(不一定要求是bp) |  |  |
| gocio.token | string[name] | accounts | 帐户信息 |  |  |
| gocio.token | GOC | stat | 代币信息 |  | |
| gocio | gocio | rammarket | ram信息 |  | |


详细说明：


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     |gocio   |proposals | 所有的提案信息 | 创建提案 | 不会删除 |

```json
{
    "rows": [
        {
            "id": 0,                    //proposals的id
            "owner": "useraaaaaaaa",    //创建该proposals的帐号
            "fee": "1000.0000 GOC",     //创建该proposals的费用
            "proposal_name": "a0",      //proposals的name
            "proposal_content": "b0",   //proposals的content
            "url": "c0",                //proposals的url
            "hash": "d0",               //proposals的hash(将用于校验proposal的信息，暂时可以随便填)
            "create_time": "2018-11-14T12:49:44",        //创建该proposal时间
            "vote_starttime": "2018-11-17T12:49:44",     //开始投票时间（需要抵押）
            "bp_vote_starttime": "2018-11-24T12:49:44",  //bp投票开始时间
            "bp_vote_endtime": "2018-12-01T12:49:44",    //bp投票结束时间，进入分奖励时间，需要由bp claimrewards触发
            "settle_time": "1970-01-01T00:00:00",        //不为0时表示已经所有投票已结束，进入计算奖励期，会计算出reward
            "reward": "0.0000 GOC",                     //对该proposal投过票的帐户能分到的奖励总和
            "total_yeas": "0.00000000000000000",        //赞成票数
            "total_nays": "0.00000000000000000",        //反对票数
            "total_voter": 0,                           //总投票数
            "bp_nays": "0.00000000000000000",           //为负数时表示赞成比反对的票数多|bp_nays|,为正数时表示反对比赞成多bp_nays,bp通过要求bp_nays<-7.0
            "total_bp": 0                               //对该proposal投过票的bp数量
        }
    ],
    "more": true
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     |int64[id]      |votes     | 对该提案的进行了投票的帐号的信息 | 提案存在，至少有一个帐号投票 |  |

```json
{
    "rows": [
        {
            "owner": "useraaaaaaaa",            //投票帐号
            "vote": 1,                          //投票内容，1赞成，0反对
            "vote_time": "2018-11-15T12:41:16", //第一次投票时间
            "vote_update_time": "2018-11-15T12:41:16" //最新投票时间
        }
    ],
    "more": true
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     |int64[id]      |bpvotes     | 对该提案的进行了投票的BP节点信息 |  |  |

```json
{
    "rows": [
        {
            "owner": "producer111a",                  //投票帐号
            "vote": 1,                                //投票内容，1赞成，0反对
            "vote_time": "2018-11-17T02:50:50",       //第一次投票时间
            "vote_update_time": "2018-11-17T02:50:50" //最新投票时间
        }
    ],
    "more": false
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     |string[name]   |gocrewards |该帐号对的投票信息以及因为投票而得到的奖励信息   |  |  |

```json
{
    "rows": [
        {
            "reward_time": "1970-01-01T00:00:00", //为0时，表示已投票，未分配奖励
            "proposal_id": 1,                     //投过票的proposal id
            "rewards": "0.0000 GOC",              //投过该proposal的奖励
            "settle_time": "1970-01-01T00:00:00"  //不为0时表示一个proposal的周期已经结束，该帐号已取回奖励
        }//reward_time是0的时候，就是已投票未分配；reward_time不为0，settle_time为0是已分配未取现；都不为0的时候，是已取现
    ],
    "more": true
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio |string[name] |gocvrewards |该帐号投票BP节点得到的奖励信息 |  |  |

```json
{
    "rows": [
        {
            "reward_time": "2018-11-15T12:42:48", //计算奖励金额的时间
            "rewards": "0.0000 GOC"          //奖励金额
        }
    ],
    "more": false
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     | string[name] |userres | 该帐号基本信息 |  |  |

```json
{
    "rows": [
        {
            "owner": "useraaaaaaaa",                  //帐号
            "net_weight": "200246162.9632 GOC",       //TODO
            "cpu_weight": "200246162.9632 GOC",
            "ram_bytes": 13673,                       //拥有的ram数量
            "governance_stake": "100000.0000 GOC",    //goc抵押数量
            "goc_stake_freeze": "2018-12-02T12:41:15" //goc抵押可以赎回时间
        }
    ],
    "more": false
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
|gocio     | gocio |global | 链的全局状态信息 |  |  |

```json
{
    "rows": [
        {//TODO
            "max_block_net_usage": 1048576,       //blockchain_parameters
            "target_block_net_usage_pct": 1000,
            "max_transaction_net_usage": 524288,
            "base_per_transaction_net_usage": 12,
            "net_usage_leeway": 500,
            "context_free_discount_net_usage_num": 20,
            "context_free_discount_net_usage_den": 100,
            "max_block_cpu_usage": 100000,
            "target_block_cpu_usage_pct": 500,
            "max_transaction_cpu_usage": 50000,
            "min_transaction_cpu_usage": 100,
            "max_transaction_lifetime": 3600,
            "deferred_trx_expiration_window": 600,
            "max_transaction_delay": 3888000,
            "max_inline_action_size": 4096,
            "max_inline_action_depth": 4,
            "max_authority_depth": 6,          //end blockchain_parameters
            "max_ram_size": "68719476736",     //eosio_global_state
            "total_ram_bytes_reserved": 806707,
            "total_ram_stake": 117410,
            "last_producer_schedule_update": "2018-11-15T13:07:52.000",
            "last_pervote_bucket_fill": "1542285768000000",
            "pervote_bucket": 18654,
            "perblock_bucket": 0,
            "total_unpaid_blocks": 3066,
            "total_activated_stake": "9981139082029",
            "thresh_activated_stake_time": "1542285759500000",
            "last_producer_schedule_size": 9,
            "total_producer_vote_weight": "19845126094666842112.00000000000000000",
            "last_name_close": "2000-01-01T00:00:00.000",
            "goc_proposal_fee_limit": 10000000,
            "goc_stake_limit": 1000000000,
            "goc_action_fee": 10000,
            "goc_max_proposal_reward": 1000000,
            "goc_governance_vote_period": 604800,
            "goc_bp_vote_period": 604800,
            "goc_vote_start_time": 259200,
            "goc_voter_bucket": 0,
            "goc_gn_bucket": 18653,
            "last_gn_bucket_empty": 1542285760    //end eosio_global_state
        }
    ],
    "more": false
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
| gocio | gocio | producers | 注册为生产者的帐号信息(不一定要求是bp) |  |  |


```json
{
    "rows": [
        {
            "owner": "producer111a",                //帐号
            "total_votes": "0.00000000000000000",   //获得的票数
            "producer_key": "GOC8imf2TDq6FKtLZ8mvXPWcd6EF2rQwo8zKdLNzsbU9EiMSt9Lwz", //帐号公钥
            "is_active": 1,
            "url": "https://producer111a.com/GOC8imf2TDq6FKtLZ8mvXPWcd6EF2rQwo8zKdLNzsbU9EiMSt9Lwz",
            "unpaid_blocks": 485,            //还未领取出块奖励的块数
            "last_claim_time": 0,            //上次claimrewards时间，两次claim 间隔至少24h
            "location": 0
        }
    ],
    "more": true
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
| gocio.token | string[name] | accounts | 帐户信息 |  |  |

```json
{
    "rows": [
        {
            "balance": "100000.0000 GOC" //帐户余额
        }
    ],
    "more": false
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
| gocio.token | GOC | stat | 代币信息 |  | | |

```json
{
    "rows": [
        {
            "supply": "1002000010.8813 GOC",
            "max_supply": "10000000000.0000 GOC",
            "issuer": "gocio"
        }
    ],
    "more": false
}
```


| code     | scope  | table    | 返回值 | 生成条件 | 删除条件 |
| -------- | ------ | -------- | ------- | ------- | -------- |
| gocio | gocio | rammarket | ram信息 |  | | |

```json
{
    "rows": [
        {
            "supply": "10000000000.0000 RAMCORE",
            "base": {
                "balance": "68718670029 RAM",
                "weight": "0.50000000000000000"
            },
            "quote": {
                "balance": "1000011.7410 GOC",
                "weight": "0.50000000000000000"
            }
        }
    ],
    "more": false
}
```