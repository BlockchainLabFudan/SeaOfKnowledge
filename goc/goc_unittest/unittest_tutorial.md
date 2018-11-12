# GOC unittest

## 综述
### unittest的作用（个人理解）
* 编写unittest是为了确保基本功能是正确的，同时也可以测试修改的代码会不会破坏原有正确的功能。
* unittest应该尽量覆盖会出错的环节。
* unittest一般不包括复杂的功能测试。

### unittest代码的位置
源代码在`$GOC/unittests/` 目录下，编译后的可执行文件为`$GOC/build/unittests/unit_test`
详细的测试内容请参见源代码中unittests目录中，其中每个文件对应一组完整测试，由BOOST_AUTO_TEST_SUITE命名，分项测试由BOOST_FIXTURE_TEST_CASE命名，EOS原生功能不在此详述。

```cpp
//此处abi_tests即为测试组名称
BOOST_AUTO_TEST_SUITE(abi_tests)

//此处abigen_double_action即为分项测试名称
BOOST_FIXTURE_TEST_CASE(abigen_double_action, abi_gen_helper)
```

使用unittests目录中的unit_test启动测试，**请注意所有的单元测试仍需事先编译完成**

```bash
#完整的一组测试
$GOC/build/unittests/unit_test -t "eosio_system_tests"
#执行某个指定的测试
$GOC/build/unittests/unit_test -t "eosio_system_tests/multiple_producer_pay"
```
测试结果显示如下：

```bash
Random number generator seeded to 1540087023
Running 1 test case...

*** No errors detected
```

## 有用的api
### get table 
```cpp
vector<char> base_tester::get_row_by_account( uint64_t code, uint64_t scope, uint64_t table, const account_name& act )

vector<char> base_tester::get_row_by_id( uint64_t code, uint64_t scope, uint64_t table, const uint64_t& id )
```
上述的code scope table 与命令cleos get table 一致，最后的参数是指根据该参数查找表中符合的数据
通过该函数可以查询之前push_action到链上的数据，并与之前的数据进行比较，就可以测试功能是否正确。

### push action
```cpp
action_result push_action( const account_name& signer, const action_name &name, const variant_object &data, bool auth = true )
```
signer是该action的发起人，name是action名，data是action内的数据。
比如要抵押,赎回,创建提案，更新提案，投票均可通过这个函数实现。



## GOC相关测试项
| 代码位置 | 测试组 | 单元测试 | 覆盖的功能 | 备注 |
| -------- | ------ | -------- | ------- | ------- |
|eosio.system_tests.cpp|eosio_system_tests|goc_stake_unstake|GOC抵押与赎回测试| 
|eosio.system_tests.cpp|eosio_system_tests|goc_new_update_prop|GOC创建与更新提案| 
|eosio.system_tests.cpp|eosio_system_tests|goc_vote_test|GOC用户对提案投票| 
|eosio.system_tests.cpp|eosio_system_tests|goc_bp_vote_test|GOC bp对提案投票| 
|eosio.system_tests.cpp|eosio_system_tests|producer_pay|GOC 投票奖励发放|在EOS的基础上修改，还未完成
|eosio.system_tests.cpp|eosio_system_tests|multiple_producer_pay|GOC 多节点投票奖励发放|在EOS的基础上修改，还未完成

### 如何编写测试代码
1、理解被测试功能的相应逻辑（重要）
2、参照已有的测试项写新的测试代码



## 参考
[1] https://www.jianshu.com/p/e07c653a7f82