---
layout:     post
title:      "eos多节点测试主网搭建"
subtitle:   "EOS笔记"
date:       2018-11-15
author:     "Liu Ke"
header-img: "img/in-post/eos.jpg"
tags:
    - EOS
---

> eos v1.2.4


## Catagory

1. [创建钱包和密钥](#创建钱包和密钥)
2. [导入私钥](#导入私钥)
3. [修改创世节点和keosd配置文件](#修改创世节点和keosd配置文件)
4. [启动nodeos节点服务](#启动nodeos节点服务)
5. [创建系统账户](#创建系统账户)
6. [部署系统合约并发行token](#部署系统合约并发行token)
7. [创建节点账户和普通账户](#创建节点账户和普通账户)
8. [注册出块节点](#注册出块节点)
9. [其他节点配置和启动](#其他节点配置和启动)
10. [投票质押赎回](#投票质押赎回)
11. [解除eosio账户特权](#解除eosio账户特权)


本文梳理一下由一个创世节点和三个出块节点启动eos测试主网，发行token，并实现出块节点轮流出块的过程。

### 创建钱包和密钥

首先在四台机器中确定一台，为创世节点。clone了eos代码成功编译安装之后，创建钱包。

```sh
$ cleos create wallet --to-console

$ cleos create key --to-console

```
然后创建密钥对，这里我创建6对密钥对，分别对应4个节点账户和2个普通账户。

### 导入私钥

将生成的6对密钥的私钥对全部导入到钱包中。

```sh
$ cleos wallet import 
```

### 修改创世节点和keosd配置文件

创世节点的`~/.local/share/eosio/nodeos/config`目录下，修改`config.ini`节点配置。通过`nodeos`命令启动节点服务之后，会在该目录下自动生成该配置文件。

node配置需要修改的参数有如下几个：

```ini
#改为本机自己的IP
bnet-endpoint = 172.18.0.1:4321

#本地节点的rpc访问地址，改成创世节点机器自己的IP，默认端口为8888
http-server-address = 172.18.0.1:8888

#bp节点间的访问地址，改成节点机器自己的IP，默认端口为9876
p2p-listen-endpoint = 172.18.0.1:9876

#节点的名字
agent-name = "EOS test Agent"
#出块的节点的账户名，创世节点必须为eosio
producer-name = eosio

#节点的私钥，创世节点不能修改私钥
signature-provider = EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF=KEY:5KbHF8P67fJ4uqti6V85nzbufVLm5D2ogFretauDyAgfQ67t95V

#创世节点必须为true
enable-stale-production = true

#添加另外三个bp节点的访问地址
p2p-peer-address =

# 可以改成一个较大的值，如3000,防止手动启动主网时报交易超时的错误
max-transaction-time = 30

#将这个参数设置为*可以保证通过get action操作获取到交易的信息
filter-on = *

#默认为1，改成false。如果设置为false，那么任何传入的“Host”报头都被认为是有效的。通过postman、getman等测试rpc接口就会正常调用。
http-validate-host = false

#添加启动节点服务时加载的插件
plugin = eosio::chain_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
plugin = eosio::producer_plugin
```

对于keosd配置文件，为`~/eos-wallet/config.ini`。keosd配置文件需要修改的：
```ini
#本地节点的钱包服务keosd地址，改成创世节点机器自己的IP，默认端口为8888，需更改端口号使之与node节点端口号不同，避免冲突。我这里改为8889
http-server-address = 172.18.0.1:8889

#默认为1，改成false。如果设置为false，那么任何传入的“Host”报头都被认为是有效的。通过postman、getman等测试钱包rpc接口就会正常调用。
http-validate-host = false

#钱包自动锁定的时间，单位为秒，可根据需要进行修改
unlock-timeout = 900
```

### 启动nodeos节点服务

使用如下命令启动nodeos，会将创世节点初始化信息保存在genesis.json文件中，其他三个节点使用这个文件来启动nodeos，能够保证网络状态信息的一致。
```sh
nodeos --extract-genesis-json genesis.json
```

### 创建系统账户


```sh
cleos --url http://172.17.0.9:8888 create account eosio eosio.bpay EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.17.0.9:8888 create account eosio eosio.msig EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.17.0.9:8888 create account eosio eosio.names EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.17.0.9:8888 create account eosio eosio.ram EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.18.0.1:8888 create account eosio eosio.ramfee EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.18.0.1:8888 create account eosio eosio.saving EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.18.0.1:8888 create account eosio eosio.stake EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.18.0.1:8888 create account eosio eosio.token EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
cleos --url http://172.18.0.1:8888 create account eosio eosio.vpay EOS6cDratMo8CQuEF4Xv1j4woaDaaiW7PagxVj3U8hckQTivq5toF
```

### 部署系统合约并发行token

```sh
cleos -u http://172.18.0.1:8888 set contract eosio.token ~/eosio/build/contracts/eosio.toke/
cleos -u http://172.18.0.1:8888 set contract eosio.msig  ~/eosio/build/contracts/eosio.msig/
cleos -u http://172.18.0.1:8888 push action eosio.token create '["eosio", "1000000000.0000 SYS",0,0,0]' -p eosio.token
cleos -u http://172.18.0.1:8888 push action eosio.token issue '["eosio", "1000000000.0000 SYS", "issue"]' -p eosio


cleos -u http://172.18.0.1:8888 set contract eosio ~/eosio/build/contracts/eosio.system/ 
cleos -u http://172.18.0.1:8888 push action eosio setpriv '["eosio.msig", 1]' -p eosio@active
```

### 创建节点账户和普通账户

我的三个节点分别命名为aaa，bbb，ccc，两个普通账户命名为user1，user2。

```sh
cleos -u http://172.18.0.1:8888 system newaccount --transfer eosio aaa EOS7V5hcYss6e89yK7W5Kq4x9iXWHM64F4VWHQdhP4TTakAJYjLgS --stake-net "1000 SYS" --stake-cpu "1000 SYS" --buy-ram "1000 SYS"   

cleos -u http://172.18.0.1:8888 system newaccount --transfer eosio bbb EOS7YQP2MxbRbSqJJ89VZJBqs44biQkzqDQj3Cxb8hVaXWZdUYpcm --stake-net "1000 SYS" --stake-cpu "1000 SYS" --buy-ram "1000 SYS"   

cleos -u http://172.18.0.1:8888 system newaccount --transfer eosio ccc EOS8b1DYUPTH6BE65jDxzqkP95PxwStoPqfV4VWHiZX5z2sR2rVJw --stake-net "1000 SYS" --stake-cpu "1000 SYS" --buy-ram "1000 SYS"   

cleos -u http://172.18.0.1:8888 system newaccount --transfer eosio user1 EOS5Vz3XzoBEjbk99yZU31cijzdLcYdADvjxWMFAQu6beUBTwdjWV --stake-net "1000 SYS" --stake-cpu "1000 SYS" --buy-ram "1000 SYS"   


cleos -u http://172.18.0.1:8888 system newaccount --transfer eosio user2 EOS6you4AMUo4K7qoLGMkTrp7qLyMe5FtpBmsSUNsVKspQhKCNqzX --stake-net "1000 SYS" --stake-cpu "1000 SYS" --buy-ram "1000 SYS"   
 
```

投票的时候会有一个问题：如果是eosio账户直接转账出来的token进行抵押投票不会改变total_activated_stake的值，但是会影响投票比率。total_activated_stake除10000及为已经投票的token数量。所以这里eosio将准备投票的token先转给user1，再由user1转给user2进行投票。
```sh
cleos -u http://172.18.0.1:8888 transfer eosio user1 "900000000.0000 SYS"

cleos -u http://172.18.0.1:8888 transfer user1 user2 "800000000.0000 SYS"

```

查看账户资金：

```sh
cleos -u http://172.18.0.1:8888 get currency balance eosio.token user2
```

获取账户和投票信息：
```sh
cleos -u http://172.18.0.1:8888 get account user2
```


### 注册出块节点

```sh
cleos -u http://172.18.0.1:8888  system regproducer aaa EOS7V5hcYss6e89yK7W5Kq4x9iXWHM64F4VWHQdhP4TTakAJYjLgS

cleos -u http://172.18.0.1:8888  system regproducer aaa EOS7YQP2MxbRbSqJJ89VZJBqs44biQkzqDQj3Cxb8hVaXWZdUYpcm

cleos -u http://172.18.0.1:8888  system regproducer aaa EOS8b1DYUPTH6BE65jDxzqkP95PxwStoPqfV4VWHiZX5z2sR2rVJw

```

查看出块节点列表：

```sh
cleos -u http://172.18.0.1:8888 system listproducers
```

### 其他节点配置和启动

另外三个节点的`config.ini`配置文件主要修改的有：

```ini
#改为本机自己的IP
bnet-endpoint = 

#本地节点的rpc访问地址，改成创世节点机器自己的IP，默认端口为8888
http-server-address = 

#bp节点间的访问地址，改成节点机器自己的IP，默认端口为9876
p2p-listen-endpoint = 

#节点的名称
agent-name = 
#节点的账户名
producer-name = 
#节点的公私钥（需要妥善保存，不能上传到github等地方）
signature-provider =
 
enable-stale-production = false
#添加除自己之外的所有节点的访问地址
p2p-peer-address = 

#添加另外三个bp节点的访问地址
p2p-peer-address =

# 可以改成一个较大的值，如3000,防止手动启动主网时报交易超时的错误
max-transaction-time = 30

#将这个参数设置为*可以保证通过get action操作获取到交易的信息
filter-on = *

#默认为1，改成false。如果设置为false，那么任何传入的“Host”报头都被认为是有效的。通过postman、getman等测试rpc接口就会正常调用。
http-validate-host = false

#添加启动节点服务时加载的插件
plugin = eosio::chain_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
plugin = eosio::producer_plugin
```

配置好之后，通过创世节点的`genesis.json`文件来分别启动这三个节点。需要注意的是，使用`genesis.json`文件启动仅限**第一次**启动，之后链正常运行时，启动节点直接用`nodeos`命令启动即可。`$GENESIS_DIR`为从创世节点拷贝过来的`genesis.json`文件所在的目录。

```sh
nodeos --genesis-json $GENESIS_DIR/genesis.json
```

### 投票质押赎回

**投票**
```sh
cleos -u http://172.18.0.1:8888  system voteproducer prods user2 aaa

cleos -u http://172.18.0.1:8888  system voteproducer prods user2 bbb

cleos -u http://172.18.0.1:8888  system voteproducer prods user2 ccc

```

投票的token数量为所有质押的token数量，包括CPU、NET和RAM资源。

**查询抵押信息**
```sh
cleos -u http://172.18.0.1:8888  system listbw user2
```

**追加抵押增加票数**
```sh
cleos -u http://172.18.0.1:8888 system delegatebw user2 aaa '100000000 SYS ' '100000000 SYS'

cleos -u http://172.18.0.1:8888 system delegatebw user2 bbb '100000000 SYS ' '100000000 SYS'

cleos -u http://172.18.0.1:8888 system delegatebw user2 ccc '100000000 SYS ' '100000000 SYS'
```

当总票数超过15%时，创世节点就会停止出块。然后三个bp节点就会开始轮流出块，主网启动。

**查看出块节点列表**

```sh
cleos -u http://172.18.0.1:8888 system listproducers
```

**查看账户及投票信息**
```sh
cleos -u http://172.18.0.1:8888 get account user2
```

**赎回抵押**

赎回抵押同时撤销相应的票数，三天后才能领取返还的token。
```sh
cleos -u http://172.18.0.1:8888 system undelegatebw user2  user2 '0.02 SYS' '0.02 SYS'
```

**领取返还的token**
```sh
cleos push action eosio refund '["本人账户名"]' -p 本人账户名
```

**查询账户token数量**

```sh
cliseat -u http://172.17.14.9:8888 get currency balance cubetrain.tk samaritan
```

### 解除eosio账户特权

主网启动之后，`eosio`账户仍有许多特权。对于我们自己的测试网络，这点可以不必考虑。但是如果是个开放的区块链项目，需要解除`eosio`账户的特权，可使用如下命令：

```sh
$ cleos push action eosio updateauth '{"account": "eosio", "permission": "owner", "parent": "", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@owner
$ cleos push action eosio updateauth '{"account": "eosio", "permission": "active", "parent": "owner", "auth": {"threshold": 1, "keys": [], "waits": [], "accounts": [{"weight": 1, "permission": {"actor": "eosio.prods", "permission": "active"}}]}}' -p eosio@active

```