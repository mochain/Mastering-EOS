# EOS环境部署
随着EOS主网上线的时间越来越近，对于超级节点竞选的话题也越来越多。很多人认为它是区块链3.0技术，可以推动区块链技术的商用落地。作为开发者，我们可以在EOS上，利用它提供的各种功能，高效地创建出区块链应用。本文以官方刚发布的EOSIO DAWN 3.0来介绍EOS的环境部署。

## 概述
在着手部署之前，我们可以先通过官方的一张EOS系统架构图，来了解一下系统中各个组件之间的关系。
![Basic-EOSIO-System-Architecture.png](https://github.com/EOSIO/eos/wiki/assets/Basic-EOSIO-System-Architecture.png)

- `nodeos` - 服务器端区块链节点组件
- `cleos` - 操作区块链和管理钱包的命令行工具
- `keosd` - 钱包管理组件

从图中的关系可见，用户可以通过 `cleos` 命令行工具对 `nodeos` 和 `keosd` 进行交互，实现对区块链和钱包的各种管理操作。

## 部署前准备
编译部署EOSIO最简单的办法是使用自动编译脚本进行编译，当然也可以自己手动编译。本文主要介绍如何使用自动编译脚本进行编译安装。

#### 目前自动编译脚本支持的系统包括：

1. Amazon 2017.09 and higher.
2. Centos 7.
3. Fedora 25 and higher (Fedora 27 recommended).
4. Mint 18.
5. Ubuntu 16.04 (Ubuntu 16.10 recommended).
6. MacOS Darwin 10.12 and higher (MacOS 10.13.x recommended).

#### 系统要求（所有平台）

- 8GB RAM free required
- 20GB Disk free required

以下我使用 `ubuntu 16.04` 操作系统进行介绍

## 获取代码
首先进入项目目录，下载源代码
```
git clone https://github.com/EOSIO/eos --recursive
```
参数 `--recursive` 表示下载项目需要的所有子模块，建议加上

## 编译安装 EOSIO
### 运行自动编译脚本
```
cd eos
./eosio_build.sh
```

如果你设备的内存或硬盘达不到最低要求，可能会出现类似下面的报错
```
ky@ubuntu:~/Project/eosio/eos$ ./eosio_build.sh

    Beginning build version: 1.2
    Fri Apr 27 08:02:24 UTC 2018
    User: ky
    git head id: 5abf3060734f2973567bbacdd3e8af7cc1fc802e
    Current branch: * master

    ARCHITECTURE: Linux

    OS name: Ubuntu
    OS Version: 16.04
    CPU speed: 2194Mhz
    CPU cores: 1
    Physical Memory: 962 Mgb
    Disk install: /dev/sda1
    Disk space total: 18G
    Disk space available: 10G
    Your system must have 7 or more Gigabytes of physical memory installed.
    Exiting now.

```

当看到下图时，代表EOS.IO已经成功的编译和安装了。
![eos_environment_deploy_1](https://user-images.githubusercontent.com/8636635/39392400-b510ffd2-4ae7-11e8-9ce0-babd6caed7d9.jpg)

### 验证环境（可选）
如果需要，你可以使用以下命令来运行一组测试程序，以便验证环境是否被正常安装。
Linux 平台：
```
~/opt/mongodb/bin/mongod -f ~/opt/mongodb/mongod.conf &
```

MacOS:
```
/usr/local/bin/mongod -f /usr/local/etc/mongod.conf &
```

然后执行：
```
cd build
make test
```

## 创建和启动单节点测试网络
编译部署好EOS环境之后，在 `build/programs/nodeos` 目录下会存在 `nodeos` 命令，可以通过运行该命令创建和启动单节点测试网络:
```
cd build/programs/nodeos
./nodeos -e -p eosio --plugin eosio::wallet_api_plugin --plugin eosio::chain_api_plugin --plugin eosio::account_history_api_plugin 
```
执行完命令之后会出现：
```
*******************************
*                             *
*   ------ NEW CHAIN ------   *
*   -  Welcome to EOSIO!  -   *
*   -----------------------   *
*                             *
*******************************

Your genesis seems to have an old timestamp
Please consider using the --genesis-timestamp option to give your genesis a recent timestamp

325596ms thread-0   producer_plugin.cpp:176       plugin_startup       ] producer plugin:  plugin_startup() end
326063ms thread-0   producer_plugin.cpp:239       block_production_loo ] eosio generated block acff902a... #1 @ 2018-04-28T09:05:26.000 with 0 trxs, lib: 0
326501ms thread-0   producer_plugin.cpp:239       block_production_loo ] eosio generated block f30acbb3... #2 @ 2018-04-28T09:05:26.500 with 0 trxs, lib: 1
327001ms thread-0   producer_plugin.cpp:239       block_production_loo ] eosio generated block 01c10821... #3 @ 2018-04-28T09:05:27.000 with 0 trxs, lib: 2
```
这时，`nodeos` 就运行起来了，但其中只有一个生产者 `eosio`

## 使用 `cleos` 与系统交互
编译部署好EOS环境之后，在 `build/programs/cleos` 目录下会存在 `cleos` 命令，可以通过运行该命令与系统进行交互。
在启动单节点测试网络之后，我们另外打开一个命令行窗口，进入 `build/programs/cleos` 目录
```
cd build/programs/cleos
```

### 查看区块链当前状态
```
> ./cleos get info
{
  "server_version": "4e99117b",
  "head_block_num": 1764,
  "last_irreversible_block_num": 1763,
  "head_block_id": "000006e432459f5588cbfba1a568bef24aa25cdfb4e460a5ef1f71ac5f8905ad",
  "head_block_time": "2018-04-28T09:59:56",
  "head_block_producer": "eosio"
}
```
信息字段说明：

- `server_version` 表示当前服务器运行的eos版本。
- `head_block_num` 表示最新区块的编号，可以理解为区块高度。
- `last_irreversible_block_num` 表示不可逆区块的最大编号。
- `head_block_id` 是最新区块的id。
- `head_block_time` 是最新区块的生成时间。
- `head_block_producer` 最新区块的生产者名称，由于我们只有一个节点，因此其取值一直是 `eosio`。

### 创建钱包
在区块链上进行各种交易操作需要用私钥签名授权，钱包就是存储这些私钥的地方。我们可以通过以下命令创建一个钱包：
```
> ./cleos wallet create
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JL9JQAAE9kY3QsCP9G3PTz5WUL5Q1d1VfVrshi82KvedimADf4"
```
命令执行完，会创建一个名为 `default` 的钱包，并返回钱包密码，该密码可以用来解锁钱包，需要保存好。

### 查看钱包列表
创建钱包之后，可以查看钱包列表：
```
> ./cleos wallet list
Wallets:
[
  "default *"
]
```
如果你没有指定钱包名称，默认操作的都是 `default` 钱包。

### 将私钥导入钱包
创建新的账户需要用到已经被初始化的账号，在测试网络中账户 `eosio` 是已经被初始化了的，可以在 `config.ini` 文件中找到它对应的公钥和私钥对:
```
private-key = ["EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"]
```
执行以下命令可以将私钥导入钱包：
```
> ./cleos wallet import 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
```

### 创建密钥对
创建新的账户还需要指定两组密钥对：owner和active。
>EOS.IO 软件给所有账户指定了两个默认权限组。一个是"owner"权限组,可以执行任何操作。还有一个“active”权限组，除了更改“owner” 权限组之外，可以执行所有操作。所有其他权限组均由“active”组派生。

可以使用以下命令创建密钥对：

- owner key
```
> ./cleos create key
Private key: 5KiUyjfLLs6HjaV2LjrXFk62AQ85d6Sfq52XgnpgPUaes1MErPT
Public key: EOS7Nc5Z4Ek8fawzrM4R2xPpEhEuEqhE6hXfLWMBHrELqstjKnAot
```
- active key
```
> ./cleos create key
Private key: 5Js2Ts5Rwdh56kLD3NDsu7ZmJxUGruQz9KFVgPwLBnJf5dJ2WbE
Public key: EOS6fCP6JxUPiK1CFvoUNnWUvkxM6UjAx2hjgHCtyjW6xeW3bPZ5G
```

### 创建账户
接下来使用 `eosio` 账户以及两组密钥对来创建新的账户 `tester`：
```
> ./cleos create account eosio tester EOS7Nc5Z4Ek8fawzrM4R2xPpEhEuEqhE6hXfLWMBHrELqstjKnAot EOS6fCP6JxUPiK1CFvoUNnWUvkxM6UjAx2hjgHCtyjW6xeW3bPZ5G
executed transaction: dbebf3334c4191c1800835b0efb07d7a07076748a6e65f27955510694d72626d  352 bytes  102400 cycles
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7Nc5Z4Ek8fawzrM4R2xPpEh...

```

### 查看账户
通过下面的命令可以查看到账户的权限等信息：
```
> ./cleos get account tester
{
  "account_name": "tester",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS6fCP6JxUPiK1CFvoUNnWUvkxM6UjAx2hjgHCtyjW6xeW3bPZ5G",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS7Nc5Z4Ek8fawzrM4R2xPpEhEuEqhE6hXfLWMBHrELqstjKnAot",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```

更多的 `cleos` 命令的使用方法可以参考 [官方Wiki](https://github.com/EOSIO/eos/wiki/Command%20Reference)