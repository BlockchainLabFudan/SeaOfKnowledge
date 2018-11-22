# todolist合约资源分析

## 合约代码

合约有一个表，表名为todos，成员有表的id，描述，是否完成三个成员。
action 有(create)(complete)(destroy)
(create) 创建一个表
(complete)标记一个表已完成
(destroy)删除一个表

```cpp
//$GOC/contracts/todolist/todolist.cpp

#include <eosiolib/eosio.hpp>
#include <eosiolib/print.hpp>

using namespace eosio;

class todolist : public eosio::contract
{
    public :
        using eosio::contract::contract;
        // @abi table todos i64
        struct todo_info
        {
            uint64_t id;
            std::string desc;
            uint64_t completed;
            
            uint64_t primary_key() const { return id; }
            EOSLIB_SERIALIZE( todo_info, (id)(desc)(completed) )
        };

    typedef eosio::multi_index<N(todos), todo_info> todo_table;
    // @abi action
    void create(account_name author, const uint64_t id, const std::string& desc)
    {
        require_auth(author);
        todo_table todos(_self, author);
        todos.emplace(author, [&](auto &new_todo) {
            new_todo.id = id;
            new_todo.desc = desc;
            new_todo.completed = 0;
        });
        eosio::print("todo#", id, " created");
    }
    // @abi action
    void complete(account_name author, const uint64_t id)
    {
        todo_table todos(_self, author);
        auto todo_lookup = todos.find(id);
        eosio_assert(todo_lookup != todos.end(), "Todo does not exist");
        todos.modify(todo_lookup, author, [&](auto& modif_todo){
            modif_todo.completed = 1;
        });

        eosio::print("todo#", id, " marked as completed");
    }
    // @abi action
    void destroy(account_name author, const uint64_t id)
    {
        todo_table todos(_self, author);
        auto todo_lookup = todos.find(id);
        todos.erase(todo_lookup);

        eosio::print("todo#", id, " destroyed");
    }
};

EOSIO_ABI(todolist, (create)(complete)(destroy))
```
## 相关命令

### 生成 abi wast wasm
```bash 
$GOC/contracts/todolist$ ./../../build/tools/eosiocpp -o todolist.wast todolist.cpp 
$GOC/contracts/todolist$ ./../../build/tools/eosiocpp -g todolist.abi todolist.cpp
```

### 创建测试帐号

本文中创建了两个帐号，分别是zmaozmaozmao，zmaozmaozmaq。第一个用来部署合约，第二个用来测试合约。

```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 system newaccount --transfer useraaaaaaaa zmaozmaozmao GOC8ZjbDEi872aLpuuAjnd76NYW6KzPaf6RBSuwXcHmKm7A1sxayV --stake-net "20.0000 GOC" --stake-cpu "20.0000 GOC" --buy-ram "20.000 GOC" 

cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 system newaccount --transfer useraaaaaaaa zmaozmaozmaq GOC8ZjbDEi872aLpuuAjnd76NYW6KzPaf6RBSuwXcHmKm7A1sxayV --stake-net "20.0000 GOC" --stake-cpu "20.0000 GOC" --buy-ram "20.000 GOC" 
```

### 部署合约命令
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 set contract zmaozmaozmao ./../../../contracts/todolist/
```

### 查看资源
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 get account -j zmaozmaozmaq

{
  "account_name": "zmaozmaozmaq",
  "head_block_num": 3786,
  "head_block_time": "2018-11-17T12:00:48.500",
  "privileged": false,
  "last_code_update": "1970-01-01T00:00:00.000",
  "created": "2018-11-17T11:35:02.000",
  "ram_quota": 1367402,
  "net_weight": 200000,
  "cpu_weight": 200000,
  "net_limit": {
    "used": 1112,   //下表中的net
    "available": 318524,
    "max": 319636
  },
  "cpu_limit": {
    "used": 5232,   //下表中的cpu
    "available": 55612,
    "max": 60844
  },
  "ram_usage": 3728,   //下表中的memory
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "GOC8ZjbDEi872aLpuuAjnd76NYW6KzPaf6RBSuwXcHmKm7A1sxayV",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "GOC8ZjbDEi872aLpuuAjnd76NYW6KzPaf6RBSuwXcHmKm7A1sxayV",
            "weight": 1
          }
        ],
        "accounts": [],
        "waits": []
      }
    }
  ],
  "total_resources": {
    "owner": "zmaozmaozmaq",
    "net_weight": "20.0000 GOC",
    "cpu_weight": "20.0000 GOC",
    "ram_bytes": 1367402,
    "governance_stake": "0.0000 GOC",
    "goc_stake_freeze": "1970-01-01T00:00:00"
  },
  "self_delegated_bandwidth": {
    "from": "zmaozmaozmaq",
    "to": "zmaozmaozmaq",
    "net_weight": "20.0000 GOC",
    "cpu_weight": "20.0000 GOC"
  },
  "refund_request": null,
  "voter_info": {
    "owner": "zmaozmaozmaq",
    "proxy": "",
    "producers": [],
    "staked": 400000,
    "last_vote_weight": "0.00000000000000000",
    "last_vote_stake": 0,
    "proxied_vote_weight": "0.00000000000000000",
    "proxied_vote_stake": 0,
    "is_proxy": 0
  }
}
```
## 测试过程


### 部署todo.list合约
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 set contract zmaozmaozmao ./../../../contracts/todolist/
```
| 帐号：zmaozmaozmao | 动作前used | 动作后used |  |
| -------- | ------ | -------- | -------- |
|memory     |3482   |106359 | +102877 |
|net |0 |5137 | +5137 |
|cpu |0 |899 | +899 |



### zmaozmaozmaq 执行create '["zmaozmaozmaq",1,"12345"]'

```bash
./cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 push action zmaozmaozmao create '["zmaozmaozmaq",1,"12345"]' -p zmaozmaozmaq
```
| 帐号：zmaozmaozmaq | 动作前used | 动作后used |      |
| ------------------ | ---------- | ---------- | ---- |
| memory             | 3482       | 3728       | +246 |
| net                | 0          | 121        | +121 |
| cpu                | 0          | 516        | +516 |

### zmaozmaozmaq 执行destroy '["zmaozmaozmaq",1]'

```bash
./cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 push action zmaozmaozmao destroy '["zmaozmaozmaq",1]' -p zmaozmaozmaq
```
| 帐号：zmaozmaozmaq | 动作前used | 动作后used |      |
| ------------------ | ---------- | ---------- | ---- |
| memory             | 3728       | 3482       | -246 |
| net                | 121        | 233        | +112 |
| cpu                | 516        | 1300       | +784 |

### zmaozmaozmaq 执行create '["zmaozmaozmaq",1,"12345678901234567890123456789012345678901234567890"]'

```bash
./cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 push action zmaozmaozmao create '["zmaozmaozmaq",1,"12345678901234567890123456789012345678901234567890"]' -p zmaozmaozmaq
```
| 帐号：zmaozmaozmaq | 动作前used | 动作后used |      |
| ------------------ | ---------- | ---------- | ---- |
| memory             | 3482       | 3773       | +291 |
| net                | 233        | 392        | +159 |
| cpu                | 1300       | 1727       | +427 |

 ### 执行查询todos表
```bash
./cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 get table zmaozmaozmao zmaozmaozmaq todos
```
| 帐号：zmaozmaozmaq | 动作前used | 动作后used |      |
| ------------------ | ---------- | ---------- | ---- |
| memory             | 3773       | 3773       | 0    |
| net                | 392        | 392        | 0    |
| cpu                | 1727       | 1727       | 0    |

### zmaozmaozmaq 执行complete '["zmaozmaozmaq",1]'

```bash
./cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 push action zmaozmaozmao complete '["zmaozmaozmaq",1]' -p zmaozmaozmaq
```
| 帐号：zmaozmaozmaq | 动作前used | 动作后used |  |
| ------ | ---------- | ---------- | ------ |
| memory | 3773 | 3773 | 0 |
| net    | 392 | 504 | +112 |
| cpu    | 1727 | 2114 | +387 |

### zmaozmaozmaq 重复执行complete '["zmaozmaozmaq",1]'

```bash
./cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8000 push action zmaozmaozmao complete '["zmaozmaozmaq",1]' -p zmaozmaozmaq
```
| 帐号：zmaozmaozmaq | 动作前used | 动作后used |  |
| ------ | ---------- | ---------- | ------ |
| memory | 3773 | 3773 | 0 |
| net    | 504 | 615 | +111 |
| cpu    | 2114 | 2587 | +473 |

以上过程帐号zmaozmaozmao一直保持着

| 帐号：zmaozmaozmao | used |
| -------- | -------- |
|memory    |106359 |
|net |5137 |
|cpu |899 |

参考

https://www.cnblogs.com/Evsward/p/multi_index.html
