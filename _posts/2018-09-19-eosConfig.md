---
layout:     post
title:      "EOS配置文件config.ini"
subtitle:   "EOS笔记"
date:       2018-09-19
author:     "Liu Ke"
header-img: "img/in-post/eos.jpg"
tags:
    - EOS
---

## cleos

如下图所示，简单来说`cleos`服务就是eos的命令行工具，通过命令行和eos进行交互并且管理钱包。`keosd`是保证钱包中密钥安全的组件，`nodeos`是后台运行的eos的主程序。

- cleos = cli + eos
- keosd = key + eos
- nodeos = node + eos

![](http://dugu0808.github.io/img/in-post/180917/cleos.png)

## config.ini

在`./eos/build/programs/nodeos`下，输入以下命令启动节点后，会在`~/.local/share/eosio/nodeos/config`目录下生成配置文件`config.ini`。需要注意的是，`~/.local`这个文件夹是隐藏文件夹，可使用`ls -a`命令查看。

```sh
$ ./nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin
```

启动节点的命令相对来说比较复杂，通过查看config.ini文件可以发现，可以通过修改如下配置项，实现直接使用`./nodeos`命令来启动。

```ini
# Enable block production, even if the chain is stale. (eosio::producer_plugin)
# 在链是stale（旧的）情况下开启区块生成
# 和启动命令中的-e对应
enable-stale-production = true

# ID of producer controlled by this node (e.g. inita; may specify multiple times) (eosio::producer_plugin)
# 由这个节点控制的生产者的ID
# 对应启动命令中 -p eosio
producer-name = eosio

# Plugin(s) to enable, may be specified multiple times
# 设置启动时要使用的插件
# 对应启动命令中 --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin 
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::http_plugin
plugin = eosio::history_api_plugin

```

同时，可以把`verbose-http-errors = false`改为`true`，可以在HTTP responses中增加错误日志，便于发现各种错误。

```ini
# Append the error log to HTTP responses (eosio::http_plugin)
# 在HTTP responses中增加错误日志
verbose-http-errors = true
```

config.ini文件内容如下，官方用英文注释了每一个配置项的功能：

```ini
# the endpoint upon which to listen for incoming connections (eosio::bnet_plugin)
bnet-endpoint = 0.0.0.0:4321

# this peer will request only irreversible blocks from other nodes (eosio::bnet_plugin)
bnet-follow-irreversible = 0

# the number of threads to use to process network messages (eosio::bnet_plugin)
# bnet-threads = 

# remote endpoint of other node to connect to; Use multiple bnet-connect options as needed to compose a network (eosio::bnet_plugin)
# bnet-connect = 

# this peer will request no pending transactions from other nodes (eosio::bnet_plugin)
bnet-no-trx = false

# The string used to format peers when logging messages about them.  Variables are escaped with ${<variable name>}.
# Available Variables:
#    _name  	self-reported name
# 
#    _id    	self-reported ID (Public Key)
# 
#    _ip    	remote IP address of peer
# 
#    _port  	remote port number of peer
# 
#    _lip   	local IP address connected to peer
# 
#    _lport 	local port number connected to peer
# 
#  (eosio::bnet_plugin)
bnet-peer-log-format = ["${_name}" ${_ip}:${_port}]

# the location of the blocks directory (absolute path or relative to application data dir) (eosio::chain_plugin)
blocks-dir = "blocks"

# Pairs of [BLOCK_NUM,BLOCK_ID] that should be enforced as checkpoints. (eosio::chain_plugin)
# checkpoint = 

# Override default WASM runtime (eosio::chain_plugin)
# wasm-runtime = 

# Override default maximum ABI serialization time allowed in ms (eosio::chain_plugin)
abi-serializer-max-time-ms = 15000

# Maximum size (in MiB) of the chain state database (eosio::chain_plugin)
chain-state-db-size-mb = 1024

# Safely shut down node when free space remaining in the chain state database drops below this size (in MiB). (eosio::chain_plugin)
chain-state-db-guard-size-mb = 128

# Maximum size (in MiB) of the reversible blocks database (eosio::chain_plugin)
reversible-blocks-db-size-mb = 340

# Safely shut down node when free space remaining in the reverseible blocks database drops below this size (in MiB). (eosio::chain_plugin)
reversible-blocks-db-guard-size-mb = 2

# print contract's output to console (eosio::chain_plugin)
contracts-console = false

# Account added to actor whitelist (may specify multiple times) (eosio::chain_plugin)
# actor-whitelist = 

# Account added to actor blacklist (may specify multiple times) (eosio::chain_plugin)
# actor-blacklist = 

# Contract account added to contract whitelist (may specify multiple times) (eosio::chain_plugin)
# contract-whitelist = 

# Contract account added to contract blacklist (may specify multiple times) (eosio::chain_plugin)
# contract-blacklist = 

# Action (in the form code::action) added to action blacklist (may specify multiple times) (eosio::chain_plugin)
# action-blacklist = 

# Public key added to blacklist of keys that should not be included in authorities (may specify multiple times) (eosio::chain_plugin)
# key-blacklist = 

# Database read mode ("speculative", "head", or "read-only").
# In "speculative" mode database contains changes done up to the head block plus changes made by transactions not yet included to the blockchain.
# In "head" mode database contains changes done up to the current head block.
# In "read-only" mode database contains incoming block changes but no speculative transaction processing.
#  (eosio::chain_plugin)
read-mode = speculative

# Chain validation mode ("full" or "light").
# In "full" mode all incoming blocks will be fully validated.
# In "light" mode all incoming blocks headers will be fully validated; transactions in those validated blocks will be trusted 
#  (eosio::chain_plugin)
validation-mode = full

# Disable the check which subjectively fails a transaction if a contract bills more RAM to another account within the context of a notification handler (i.e. when the receiver is not the code of the action). (eosio::chain_plugin)
disable-ram-billing-notify-checks = false

# Track actions which match receiver:action:actor. Actor may be blank to include all. Action and Actor both blank allows all from Recieiver. Receiver may not be blank. (eosio::history_plugin)
# filter-on = 

# Do not track actions which match receiver:action:actor. Action and Actor both blank excludes all from Reciever. Actor blank excludes all from reciever:action. Receiver may not be blank. (eosio::history_plugin)
# filter-out = 

# PEM encoded trusted root certificate (or path to file containing one) used to validate any TLS connections made.  (may specify multiple times)
#  (eosio::http_client_plugin)
# https-client-root-cert = 

# true: validate that the peer certificates are valid and trusted, false: ignore cert errors (eosio::http_client_plugin)
https-client-validate-peers = 1

# The local IP and port to listen for incoming http connections; set blank to disable. (eosio::http_plugin)
http-server-address = 127.0.0.1:8888

# The local IP and port to listen for incoming https connections; leave blank to disable. (eosio::http_plugin)
# https-server-address = 

# Filename with the certificate chain to present on https connections. PEM format. Required for https. (eosio::http_plugin)
# https-certificate-chain-file = 

# Filename with https private key in PEM format. Required for https (eosio::http_plugin)
# https-private-key-file = 

# Specify the Access-Control-Allow-Origin to be returned on each request. (eosio::http_plugin)
# access-control-allow-origin = 

# Specify the Access-Control-Allow-Headers to be returned on each request. (eosio::http_plugin)
# access-control-allow-headers = 

# Specify the Access-Control-Max-Age to be returned on each request. (eosio::http_plugin)
# access-control-max-age = 

# Specify if Access-Control-Allow-Credentials: true should be returned on each request. (eosio::http_plugin)
access-control-allow-credentials = false

# The maximum body size in bytes allowed for incoming RPC requests (eosio::http_plugin)
max-body-size = 1048576

# Append the error log to HTTP responses (eosio::http_plugin)
# 在HTTP responses中增加错误日志
verbose-http-errors = false

# If set to false, then any incoming "Host" header is considered valid (eosio::http_plugin)
http-validate-host = 1

# Additionaly acceptable values for the "Host" header of incoming HTTP requests, can be specified multiple times.  Includes http/s_server_address by default. (eosio::http_plugin)
# http-alias = 

# The maximum number of pending login requests (eosio::login_plugin)
max-login-requests = 1000000

# The maximum timeout for pending login requests (in seconds) (eosio::login_plugin)
max-login-timeout = 60

# The target queue size between nodeos and MongoDB plugin thread. (eosio::mongo_db_plugin)
mongodb-queue-size = 1024

# The maximum size of the abi cache for serializing data. (eosio::mongo_db_plugin)
mongodb-abi-cache-size = 2048

# Required with --replay-blockchain, --hard-replay-blockchain, or --delete-all-blocks to wipe mongo db.This option required to prevent accidental wipe of mongo db. (eosio::mongo_db_plugin)
mongodb-wipe = false

# If specified then only abi data pushed to mongodb until specified block is reached. (eosio::mongo_db_plugin)
mongodb-block-start = 0

# MongoDB URI connection string, see: https://docs.mongodb.com/master/reference/connection-string/. If not specified then plugin is disabled. Default database 'EOS' is used if not specified in URI. Example: mongodb://127.0.0.1:27017/EOS (eosio::mongo_db_plugin)
# mongodb-uri = 

# Enables storing blocks in mongodb. (eosio::mongo_db_plugin)
mongodb-store-blocks = 1

# Enables storing block state in mongodb. (eosio::mongo_db_plugin)
mongodb-store-block-states = 1

# Enables storing transactions in mongodb. (eosio::mongo_db_plugin)
mongodb-store-transactions = 1

# Enables storing transaction traces in mongodb. (eosio::mongo_db_plugin)
mongodb-store-transaction-traces = 1

# Enables storing action traces in mongodb. (eosio::mongo_db_plugin)
mongodb-store-action-traces = 1

# Mongodb: Track actions which match receiver:action:actor. Actor may be blank to include all. Receiver and Action may not be blank. Default is * include everything. (eosio::mongo_db_plugin)
# mongodb-filter-on = 

# Mongodb: Do not track actions which match receiver:action:actor. Action and Actor both blank excludes all from reciever. Actor blank excludes all from reciever:action. Receiver may not be blank. (eosio::mongo_db_plugin)
# mongodb-filter-out = 

# The actual host:port used to listen for incoming p2p connections. (eosio::net_plugin)
p2p-listen-endpoint = 0.0.0.0:9876

# An externally accessible host:port for identifying this node. Defaults to p2p-listen-endpoint. (eosio::net_plugin)
# p2p-server-address = 

# The public endpoint of a peer node to connect to. Use multiple p2p-peer-address options as needed to compose a network. (eosio::net_plugin)
# p2p-peer-address = 

# Maximum number of client nodes from any single IP address (eosio::net_plugin)
p2p-max-nodes-per-host = 1

# The name supplied to identify this node amongst the peers. (eosio::net_plugin)
agent-name = "EOS Test Agent"

# Can be 'any' or 'producers' or 'specified' or 'none'. If 'specified', peer-key must be specified at least once. If only 'producers', peer-key is not required. 'producers' and 'specified' may be combined. (eosio::net_plugin)
allowed-connection = any

# Optional public key of peer allowed to connect.  May be used multiple times. (eosio::net_plugin)
# peer-key = 

# Tuple of [PublicKey, WIF private key] (may specify multiple times) (eosio::net_plugin)
# peer-private-key = 

# Maximum number of clients from which connections are accepted, use 0 for no limit (eosio::net_plugin)
max-clients = 25

# number of seconds to wait before cleaning up dead connections (eosio::net_plugin)
connection-cleanup-period = 30

# max connection cleanup time per cleanup call in millisec (eosio::net_plugin)
max-cleanup-time-msec = 10

# True to require exact match of peer network version. (eosio::net_plugin)
network-version-match = 0

# number of blocks to retrieve in a chunk from any individual peer during synchronization (eosio::net_plugin)
sync-fetch-span = 100

# maximum sizes of transaction or block messages that are sent without first sending a notice (eosio::net_plugin)
max-implicit-request = 1500

# Enable expirimental socket read watermark optimization (eosio::net_plugin)
use-socket-read-watermark = 0

# The string used to format peers when logging messages about them.  Variables are escaped with ${<variable name>}.
# Available Variables:
#    _name  	self-reported name
# 
#    _id    	self-reported ID (64 hex characters)
# 
#    _sid   	first 8 characters of _peer.id
# 
#    _ip    	remote IP address of peer
# 
#    _port  	remote port number of peer
# 
#    _lip   	local IP address connected to peer
# 
#    _lport 	local port number connected to peer
# 
#  (eosio::net_plugin)
peer-log-format = ["${_name}" ${_ip}:${_port}]

# Enable block production, even if the chain is stale. (eosio::producer_plugin)
# 在链是stale（旧的）情况下开启区块生成
enable-stale-production = false

# Start this node in a state where production is paused (eosio::producer_plugin)
pause-on-startup = false

# Limits the maximum time (in milliseconds) that is allowed a pushed transaction's code to execute before being considered invalid (eosio::producer_plugin)
max-transaction-time = 30

# Limits the maximum age (in seconds) of the DPOS Irreversible Block for a chain this node will produce blocks on (use negative value to indicate unlimited) (eosio::producer_plugin)
max-irreversible-block-age = -1

# ID of producer controlled by this node (e.g. inita; may specify multiple times) (eosio::producer_plugin)
# producer-name = 

# (DEPRECATED - Use signature-provider instead) Tuple of [public key, WIF private key] (may specify multiple times) (eosio::producer_plugin)
# private-key = 

# Key=Value pairs in the form <public-key>=<provider-spec>
# Where:
#    <public-key>    	is a string form of a vaild EOSIO public key
# 
#    <provider-spec> 	is a string in the form <provider-type>:<data>
# 
#    <provider-type> 	is KEY, or KEOSD
# 
#    KEY:<data>      	is a string form of a valid EOSIO private key which maps to the provided public key
# 
#    KEOSD:<data>    	is the URL where keosd is available and the approptiate wallet(s) are unlocked (eosio::producer_plugin)
signature-provider = EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV=KEY:5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3

# Limits the maximum time (in milliseconds) that is allowd for sending blocks to a keosd provider for signing (eosio::producer_plugin)
keosd-provider-timeout = 5

# account that can not access to extended CPU/NET virtual resources (eosio::producer_plugin)
# greylist-account = 

# offset of non last block producing time in micro second. Negative number results in blocks to go out sooner, and positive number results in blocks to go out later (eosio::producer_plugin)
produce-time-offset-us = 0

# offset of last block producing time in micro second. Negative number results in blocks to go out sooner, and positive number results in blocks to go out later (eosio::producer_plugin)
last-block-time-offset-us = 0

# ratio between incoming transations and deferred transactions when both are exhausted (eosio::producer_plugin)
incoming-defer-ratio = 1

# Lag in number of blocks from the head block when selecting the reference block for transactions (-1 means Last Irreversible Block) (eosio::txn_test_gen_plugin)
txn-reference-block-lag = 0

# Plugin(s) to enable, may be specified multiple times
# 设置启动时要使用的插件
# plugin = 

```
