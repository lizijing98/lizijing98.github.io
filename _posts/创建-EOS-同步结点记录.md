---
title: 创建 EOS 同步结点记录
date: 2022-03-01 09:49:55
update: 2022-03-01 09:49:55
categories: 
  - [学习,区块链]
tags: 
  - 区块链
  - EOS
  - 同步结点
cover: https://eos.io/wp-content/uploads/2020/12/about-hero-graphic.png
typora-copy-images-to: upload
---

# 创建 EOS 同步结点记录

## 1. 准备工作

本地安装 EOSIO 套件，并启动一个 BP 结点。

```shell
nodeos \
-e -p eosio \
--data-dir ./data \
--config-dir .  \
--delete-all-blocks \
>> prod.log 2>&1 &
```

config.ini：

```ini
# print contract's output to console (eosio::chain_plugin)
contracts-console = true

# The local IP and port to listen for incoming http connections; set blank to disable. (eosio::http_plugin)
http-server-address = 0.0.0.0:8888

# The actual host:port used to listen for incoming p2p connections. (eosio::net_plugin)
p2p-listen-endpoint = 0.0.0.0:9876

# Specify the Access-Control-Allow-Origin to be returned on each request. (eosio::http_plugin)
access-control-allow-origin = '*'

# Append the error log to HTTP responses (eosio::http_plugin)
verbose-http-errors = true

# If set to false, then any incoming "Host" header is considered valid (eosio::http_plugin)
http-validate-host = false

# Additionaly acceptable values for the "Host" header of incoming HTTP requests, can be specified multiple times.  Includes http/s_server_address by default. (eosio::http_plugin)
http-alias = ["localhost","127.0.0.1"]

# the location of the state-history directory (absolute path or relative to application data dir) (eosio::state_history_plugin)
state-history-dir = "/shpdata"

# enable trace history (eosio::state_history_plugin)
trace-history = true

# enable chain state history (eosio::state_history_plugin)
chain-state-history = true

# Plugin(s) to enable, may be specified multiple times
plugin = eosio::http_plugin
plugin = eosio::net_plugin
plugin = eosio::chain_plugin
plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::producer_api_plugin
```

## 2. 创建同步结点

同步结点配置文件在生产结点配置文件的基础上修改相应服务端口，并添加生产结点 p2p 服务地址：

```ini
# The public endpoint of a peer node to connect to. Use multiple p2p-peer-address options as needed to compose a network.
#   Syntax: host:port[:<trx>|<blk>]
#   The optional 'trx' and 'blk' indicates to node that only transactions 'trx' or blocks 'blk' should be sent.  Examples:
#     p2p.eos.io:9876
#     p2p.trx.eos.io:9876:trx
#     p2p.blk.eos.io:9876:blk
#  (eosio::net_plugin)
# 0.0.0.0:9876 为生产结点 p2p 地址
p2p-peer-address = 0.0.0.0:9876
```

启动同步结点命令：

```shell
nodeos \
--data-dir ./data \
--config-dir .  \
--delete-all-blocks \
>> lis.log 2>&1 &
```

相比生产结点的启动命令，删除了开启出块和标记出块者的配置

启动后可以看到结点已经开始同步：

![image-20220301102522239](https://s2.loli.net/2022/03/01/B79SywXYUoFnkvK.png)

## 3. 使用快照同步结点

## 4. 使用主网快照同步
