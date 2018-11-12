#GOC系统级智能合约笔记
GOC系统级合约有  **eosio.system** , **eosio.bios** , **eosio.msig** , **eosio.sudo** , **eosio.token** 5个在链运转前安装基础性合约，提供基于链的基本操作。

##eosio.system
**eosio.system**作为EOS最重要的智能合约，主要提供了三个核心功能 ： 

* 用户抵押token，可以投票给区块生成者（block producer），还有获得社区提案（worker proposal）的权利。
* 设置代理，把投票权移交给其他用户。
* 抵押token，获得相应的网络带宽，存储空间，运算能力。

###eosiosystem::system_contract类中定义的结构体

* name_bid 名字拍卖的结构体

```c++
   struct name_bid {
     account_name            newname;
     account_name            high_bidder;
     int64_t                 high_bid = 0; ///< negative high_bid == closed auction waiting to be claimed
     uint64_t                last_bid_time = 0;

     auto     primary_key()const { return newname;                          }
     uint64_t by_high_bid()const { return static_cast<uint64_t>(-high_bid); }
   };
```

* eosio_global_state , 定义了全局状态

```c++
struct eosio_global_state : eosio::blockchain_parameters {
      uint64_t free_ram()const { return max_ram_size - total_ram_bytes_reserved; }

      uint64_t             max_ram_size = 64ll*1024 * 1024 * 1024;
      uint64_t             total_ram_bytes_reserved = 0;
      int64_t              total_ram_stake = 0;

      block_timestamp      last_producer_schedule_update;
      uint64_t             last_pervote_bucket_fill = 0;
      int64_t              pervote_bucket = 0;
      int64_t              perblock_bucket = 0;
      uint32_t             total_unpaid_blocks = 0; /// all blocks which have been produced but not paid
      int64_t              total_activated_stake = 0;
      uint64_t             thresh_activated_stake_time = 0;
      uint16_t             last_producer_schedule_size = 0;
      double               total_producer_vote_weight = 0; /// the sum of all producer votes
      block_timestamp      last_name_close;

      //GOC parameters
      int64_t              goc_proposal_fee_limit=  10000000;
      int64_t              goc_stake_limit = 1000000000;
      int64_t              goc_action_fee = 10000;
      int64_t              goc_max_proposal_reward = 1000000;
      uint32_t             goc_governance_vote_period = 24 * 3600 * 7;  // 7 days
      uint32_t             goc_bp_vote_period = 24 * 3600 * 7;  // 7 days
      uint32_t             goc_vote_start_time = 24 * 3600 * 3;  // vote start after 3 Days
      
      int64_t              goc_voter_bucket = 0;
      int64_t              goc_gn_bucket = 0;
      uint32_t             last_gn_bucket_empty = 0;
    };
```

* produce_info 定义了生产者信息

```c++
struct producer_info {
      account_name          owner;
      double                total_votes = 0;
      eosio::public_key     producer_key; /// a packed public key object
      bool                  is_active = true;
      std::string           url;
      uint32_t              unpaid_blocks = 0;
      uint64_t              last_claim_time = 0;
      uint16_t              location = 0;

      uint64_t primary_key()const { return owner;                                   }
      double   by_votes()const    { return is_active ? -total_votes : total_votes;  }
      bool     active()const      { return is_active;                               }
      void     deactivate()       { producer_key = public_key(); is_active = false; }
   };
```

* voter_info 定义了投票人信息

```c++
struct voter_info {
      account_name                owner = 0; /// the voter
      account_name                proxy = 0; /// the proxy set by the voter, if any
      std::vector<account_name>   producers; /// the producers approved by this voter if no proxy set
      int64_t                     staked = 0;

      /**
       *  Every time a vote is cast we must first "undo" the last vote weight, before casting the
       *  new vote weight.  Vote weight is calculated as:
       *
       *  stated.amount * 2 ^ ( weeks_since_launch/weeks_per_year)
       */
      double                      last_vote_weight = 0; /// the vote weight cast the last time the vote was updated

      /**
       * Total vote weight delegated to this voter.
       */
      double                      proxied_vote_weight= 0; /// the total vote weight delegated to this voter as a proxy
      bool                        is_proxy = 0; /// whether the voter is a proxy for others


      uint32_t                    reserved1 = 0;
      time                        reserved2 = 0;
      eosio::asset                reserved3;

      uint64_t primary_key()const { return owner; }
   };
```
* goc_proposal _ info 定义了抵押过GOC token的用户提交propsal的信息

```c++
struct goc_proposal_info {
      uint64_t              id;
      account_name          owner;
      asset                 fee;
      std::string           proposal_name;
      std::string           proposal_content;
      std::string           url;

      time                  create_time;
      time                  vote_starttime;
      time                  bp_vote_starttime;
      time                  bp_vote_endtime;

      time                  settle_time = 0;
      asset                 reward;

      double                total_yeas;
      double                total_nays;
      uint64_t              total_voter = 0;
      
      double                bp_nays;
      uint16_t              total_bp = 0;

      uint64_t  primary_key()const     { return id; }
      uint64_t  by_endtime()const      { return bp_vote_endtime; }
      bool      vote_pass()const       { return total_yeas > total_nays;  }
      //need change to bp count
      bool      bp_pass()const         { return bp_nays < -7.0;  }
   };
```

* goc_vote _info 定义了proposal投票的信息

```c++
struct goc_vote_info {
     account_name           owner;
     bool                   vote;
     time                   vote_time;
     time                   vote_update_time;
     time                   settle_time = 0;

     uint64_t primary_key()const { return owner; }
   };

```
* goc_reward _info 对于BP出块的奖励信息

```c++
struct goc_reward_info {
      time          reward_time;
      uint64_t      proposal_id;
      eosio::asset  rewards = asset(0);

      uint64_t  primary_key()const { return proposal_id; }
     };
```

###在delegate_bandwidth.cpp中实现的方法

* ***delegatebw函数*** ：用于用户抵押token，获取cpu和贷款资源。

```c++
void delegatebw( account_name from, account_name receiver,
                asset stake_net_quantity, asset stake_cpu_quantity, 
                bool transfer );
```

在delegated_bandwidth.hpp中，其内部调用了

```c++
void system_contract::changebw( account_name from, account_name receiver,
               const asset stake_net_delta, const asset stake_cpu_delta, bool transfer )；
```

from : 从哪个账号扣除用来抵押的代币

receiver : 抵押的代币的接受者，表示抵押获取的资源作用在哪个账号上

stake_net_quantity : 用来抵押带宽资源的代币数量

stake_cpu_quantity : 用来抵押计算资源的代币数量

transfer : 是否接受者可以主动解除抵押获得代币，如果不是，只有发起者能够解除抵押收回代币

* ***undelegatebw函数*** ：用来解除抵押，释放资源，收回代币

```c++
void system_contract::undelegatebw( account_name from, account_name receiver,
                    asset unstake_net_quantity, asset unstake_cpu_quantity )；
```
内部调用函数：

```c++
changebw( from, receiver, -unstake_net_quantity, -unstake_cpu_quantity, false);
```
from : 解除用哪个账号所抵押的代币

receiver : 解除作用在哪个账号上的抵押代币

unstake_net_quantity : 解除用来获取带宽资源的代币数量

unstake_cpu_quantity : 解除用来获取计算资源的代币数量

* ***buyram函数***和***buyrambytes函数*** ：购买存储资源，区别是买特定数量的代币还是特定大小的内存。

```c++
void system_contract::buyram( account_name payer, account_name receiver, asset quant );

void system_contract::buyrambytes( account_name payer, account_name receiver, uint32_t bytes );
```
buyer : 购买存储资源的账号

receiver : 接受存储资源的账号

tokens : 购买存储资源所用的代币数量

bytes : 都买存储资源空间大小的数值

* ***sellram函数***：出售不需要的存储资源,出售后资源会马上释放，收入的代币也会马上入账。

```c++
void system_contract::sellram( account_name account, int64_t bytes );
```
receiver : 出售资源代币的接受账号

bytes : 出售多少空间的存储资源

* ***refund函数***：在***undelegateb函数***调用解除代币抵押后，将抵押的代币退回账户，会有个缓冲时间。

```c++
void system_contract::refund( const account_name owner );
```

###在voting.cpp中实现的方法

* ***regproducer函数***：注册成为超级节点候选人，注册后可以接受用户投票。

```c++
void system_contract::regproducer( const account_name producer, const eosio::public_key& producer_key, const std::string& url, uint16_t location ) {
      eosio_assert( url.size() < 512, "url too long" );
```
producer : 候选节点的账户名

producer_key : 候选节点的账户公钥

url : 候选节点的网站地址

location : 候选节点的机房地理位置

* ***unregprod函数***：取消成为超级节点候选人。

```c++
void system_contract::unregprod( const account_name producer );
```

* ***voteproducer函数***：抵押过token的用户给候选超级节点投票,可以委托给代理人投票，也可以自己直接投票给多个超级节点（少于30个）；

```c++
void system_contract::voteproducer( const account_name voter_name, const account_name proxy, const std::vector<account_name>& producers ) {
      require_auth( voter_name );
      update_votes( voter_name, proxy, producers, true );
   }
```
voter : 投票人

proxy : 代理投票人

producers : 得票人列表

内部调用了核心方法：

```c++
oid system_contract::update_votes( const account_name voter_name, const account_name proxy, const std::vector<account_name>& producers, bool voting );
```

* ***regproxy函数***： 注册成为投票代理人，接受其他用户的委托。

```c++
void system_contract::regproxy( const account_name proxy, bool isproxy );
```

proxy : 申请成为代理人的账号。

isproxy : 已经使用了代理投票的人，isproxy = 1 ， 不可以成为代理人。


###在producer_pay.cpp中实现的方法

* ***onblock*** ： 计算一些遗漏的区块，更新指定生产者的区块信息，每次生产区块就会执行。
onlock函数在每次生产者出块的时候都会被调用，见证者收到区块后也会调用(相当于验证区块)，每次生产者出块都会把该生产者的出块数进行统计，把所有的区块也进行统计（网上说法，还是不太懂）。

```c++
void system_contract::onblock( block_timestamp timestamp, account_name producer );
```

* ***claimrewards*** ： 生产者获得出块奖励。

```c++
void system_contract::claimrewards( const account_name& owner );
```




###在eosio.system.cpp中实现的方法
* ***setram函数***：设置整个链的最大ram，只可以增加，不可以减少。

```c++
void system_contract::setram( uint64_t max_ram_size );
```

* ***setparams***  //TODO不知道。。。



```c++
void system_contract::setparams( const eosio::blockchain_parameters& params ) {
      require_auth( N(gocio) );
      (eosio::blockchain_parameters&)(_gstate) = params;
      eosio_assert( 3 <= _gstate.max_authority_depth, "max_authority_depth should be at least 3" );
      set_blockchain_parameters( params );
   }
```

```c++
void system_contract::setparams( const eosio::blockchain_parameters& params );
```

* ***setpriv*** : 为账户设置特殊权限（）

```c++
void system_contract::setpriv( account_name account, uint8_t ispriv ) ;
```
* ***rmvproducer函数*** : 将某个超级节点移出。

```c++
void system_contract::rmvproducer( account_name producer );
```
* ***bindname函数*** :

```c++
void system_contract::bidname( account_name bidder, account_name newname, asset bid );
```
* ***update _ elected _ producer函数*** ： 更具票选的排名，更新超级节点。

```c++
void system_contract::update_elected_producers( block_timestamp block_time );
```



## 在governance.cpp中实现的GOC治理方法



* ***gocstake函数***：提出提议前需要抵押一定数量的GOC，

```c++
void system_contract::gocstake(account_name payer);
```

* ***gocunstake*** ：用户取回抵押的GOC。

```c++
void system_contract::gocunstake(account_name receiver)
```

* ***gocnewprop***：提出一个提议，对应终端的createproposal。

```c++
void system_contract::gocnewprop(
    const account_name owner, 
    asset fee, 
    const std::string &pname, 
    const std::string &pcontent,
    const std::string &url,
    uint16_t start_type)
```

owner:   提议的拥有者。
fee ：创建提议所需要的抵押的GOC                  
name：提案的名字              
content ：创建提案的文本信息 
url：创建的提案的url 
--start-type UINT  : Set start type, 1 for start vote, 2 for start bp vote(DEBUG)

* ***gocupprop函数*** : 对一个提议进行更新,对应终端的updateproposal命令。

```c++
void system_contract::gocupprop(
    const account_name owner, 
    uint64_t id, const std::string &pname, 
    const std::string &pcontent, 
    const std::string &url)
```

owner TEXT                  The owner of updating proposal (required)

 id TEXT                     The id of updating proposal (required)

 name TEXT                   The name of updating proposal (required)

 content TEXT                The content of updating proposal (required)

 url TEXT                    The url of updating proposal (required)

* ***gocsetpstage函数*** ：对提议的状态进行更新,cleos的setpstage命令。

```c++
void system_contract::gocsetpstage(
    const account_name owner, 
    uint64_t id, 
    uint16_t stage, 
    time start_time)
```

 owner TEXT                  The owner of updating proposal (required)

 id TEXT                     The id of updating proposal (required)

 stage TEXT                  To which stage (0-4), 0:new, 1:voting, 2:bp voting, 3:ended, 4:settled (required)

 starttime TEXT              Stage start time, Only for testnet

* ***gocvote函数***： 治理节点对提案进行投票。(如果一半的人不参与怎么办)

```
void system_contract::gocvote(account_name voter, uint64_t pid, bool yea)
```

owner ：投票人

pid ：投票的提议id

yea ：同意或者不同意.                   

* ***gocbpvote函数*** ：bp节点最后对提案进行投票

```c++
void system_contract::gocbpvote(account_name voter, uint64_t pid, bool yea)
```

owner ：投票人

pid ：投票的提议id

yea ：同意或者不同意. 