---
title: Mac 环境下单主机创建 EOS 单 BP 节点测试网记录
data: 2022-02-24 22:45:35
update: 2022-02-28 18:03:54
categories: 
  - [学习,区块链]
tags: 
  - 区块链
  - EOS
  - Mac
  - 单 BP 结点
cover: https://eos.io/wp-content/uploads/2020/12/about-hero-graphic.png
---

# Mac 环境下单主机创建 EOS 单 BP 节点测试网记录

## 1. eosio 套件安装

eosio 套件安装有多种方式，Mac 环境可以直接通过 Homebrew 进行安装：

```shell
brew tap eosio/eosio
brew install eosio
```

eosio 的前置依赖包括：gmp , libpqxx , libusb , openssl@1.1，如果通过 Homebrew 安装报依赖错误可以先安装依赖再进行安装。

官方给出的在不同环境下的安装方法：[Install Prebuilt Binaries | EOSIO Developer Docs](https://developers.eos.io/manuals/eos/latest/install/install-prebuilt-binaries)

或者可以拉取源码本地编译：[Build From Source | EOSIO Developer Docs](https://developers.eos.io/manuals/eos/latest/install/build-from-source/index)

**这里注意！通过 Homebrew 安装的 eosio 会有一点小坑！**

安装完成后可以在命令行输入 `nodeos -v` 进行测试：

![image-20220224214426345](https://s2.loli.net/2022/02/24/uZeGMfE372kQIKY.png)

## 2. 创建单 BP 结点测试网

单主机单 BP 结点测试网示意图：

![单主机单 BP 结点测试网示意图](https://developers.eos.io/315123127612b3c9153341b9e7401d02/single-host-single-node-testnet.png)

### 2.1 启动生产结点

官方给出的命令如下：

```shell
nodeos -e -p eosio --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin
```

命令参数说明：

- `-e`：enabling block production，开启出块
- `-p eosio`：identifying itself as block producer "eosio"，标注出块生产者为 eosio
- `--plugin eosio::chain_api_plugin`：loading chain_api_plugin，加载 chain_api_plugin 插件
- `--plugin eosio::history_api_plugin`：loading chain_api_plugin，加载 history_api_plugin 插件

此时就可以看到已经在出块了：

![image-20220224221422165](https://s2.loli.net/2022/02/24/q5ZhejpAgSYkP84.png)

通过 get_info 查看链信息：

```shell
curl http://127.0.0.1:8888/v1/chain/get_info
{"server_version":"26a4d285","chain_id":"8a34ec7df1b8cd06ff4a8abbaa7cc50300823350cadc59ab296cb00d104d2b8f","head_block_num":66,"last_irreversible_block_num":65,"last_irreversible_block_id":"0000004161687b87e669cc82fdef3cc47a1d53c5b9414bb1c2212f437cdcf817","head_block_id":"00000042de6419d46c4b7ffad7169abf945ebac391366cb1246e93aeef3af004","head_block_time":"2022-02-24T14:15:16.000","head_block_producer":"eosio","virtual_block_cpu_limit":213407,"virtual_block_net_limit":1118998,"block_cpu_limit":199900,"block_net_limit":1048576,"server_version_string":"v2.1.0","fork_db_head_block_num":66,"fork_db_head_block_id":"00000042de6419d46c4b7ffad7169abf945ebac391366cb1246e93aeef3af004","server_full_version_string":"v2.1.0-26a4d285d0be1052d962149e431eb81500782991","last_irreversible_block_time":"2022-02-24T14:15:15.500"}
```

观察启动日志可以看到：

```shell
info  2022-02-24T14:13:03.789 thread-0  main.cpp:138                  main                 ] nodeos version v2.1.0 v2.1.0-26a4d285d0be1052d962149e431eb81500782991
info  2022-02-24T14:13:03.789 thread-0  main.cpp:139                  main                 ] nodeos using configuration file /Users/d3j/Library/Application Support/eosio/nodeos/config/config.ini
info  2022-02-24T14:13:03.790 thread-0  main.cpp:140                  main                 ] nodeos data directory is /Users/d3j/Library/Application Support/eosio/nodeos/data
```

Mac 环境下 eosio 的数据默认存储位置为：`~/Library/Application Support/eosio/nodeos/data`

Linux 环境下 eosio 的数据默认存储位置为：`~/.local/share/eosio/nodeos/data`

下面就介绍如何自定配置进行创建结点

### 2.2 自定配置进行创建结点

#### 2.2.1 通过命令行指定配置

```shell
nodeos \
  -e -p eosio \
  --data-dir /users/mydir/eosio/data     \
  --config-dir /users/mydir/eosio/config \
  --plugin eosio::producer_plugin      \
  --plugin eosio::chain_plugin         \
  --plugin eosio::http_plugin          \
  --plugin eosio::state_history_plugin \
  --contracts-console   \
  --disable-replay-opts \
  --access-control-allow-origin='*' \
  --http-validate-host=false        \
  --verbose-http-errors             \
  --state-history-dir /shpdata \
  --trace-history              \
  --chain-state-history        \
  >> nodeos.log 2>&1 &
```

这是官方给出的一个起结点命令，参数说明：

- `--data-dir /users/mydir/eosio/data` 指定区块数据存储目录
- `--config-dir /users/mydir/eosio/config` 指定配置文件 `config.ini` 存储目录
- `--contracts-console` 插件 chain_plugin 中的配置，在控制台中输出合约相关信息
- `disable-replay-opts` 插件 chain_plugin 中的配置，禁用专门针对replay的优化
- `--access-control-allow-origin='*'` 插件 http_plugin 中的配置，指定允许在每次请求时返回原点的控制
- `--http-validate-host=false` 插件 http_plugin 中的配置，如果设置为false，则任何传入的 "host" 头都被视为有效
- `--verbose-http-errors` 插件 http_plugin 中的配置，将错误日志附加到 HTTP 响应
- `--state-history-dir /shpdata` 插件 state_history_plugin 中的配置，指定状态历史的目录
- `--trace-history` 插件 state_history_plugin 中的配置，启用跟踪历史
- `--chain-state-history` 插件 state_history_plugin 中的配置，启用链状态历史

所有的插件配置都可在 EOS 开发者文档中找到说明：[Plugins | EOSIO Developer Docs](https://developers.eos.io/manuals/eos/v2.2/nodeos/plugins/index)

#### 2.2.2 通过 config.ini 指定配置

通过命令行指定配置并不方便，因此可以使用配置文件管理部分插件的配置：`config.ini`

Mac 环境下默认的 `config.ini` 位置在：`~/Library/Application Support/eosio/nodeos/config`

Linux 环境默认的 `config.ini` 位置在：`~/.local/share/eosio/nodeos/config`

关于配置文件中对应内容可以参照默认配置文件中的注释，也可以结合官方文档中插件的配置说明进行查看：[Plugins | EOSIO Developer Docs](https://developers.eos.io/manuals/eos/latest/nodeos/plugins/index)

**注意，部分插件的配置只能通过命令行启动时进行指定**

例如现在有一个配置文件如下：

```ini
# print contract's output to console (eosio::chain_plugin)
contracts-console = true

# The local IP and port to listen for incoming http connections; set blank to disable. (eosio::http_plugin)
http-server-address = 0.0.0.0:8888

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

通过该配置文件起单 BP 结点的命令如下：

```shell
nodeos \
-e -p eosio \
--data-dir ./data \
--config-dir . \
--delete-all-blocks
```

执行后可见正常出块，测试配置文件是否生效，例如测试 `http-alias` ：

![image-20220228180257064](https://s2.loli.net/2022/02/28/2k5hSVGD3gcCnbW.png)
