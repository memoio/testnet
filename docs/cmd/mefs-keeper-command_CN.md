# MEFS 命令行操作文档

[TOC]

## 命令详情

### 启动

#### 初始化

mefs 初始化，默认初始化的目录为\$HOME/.mefs，可以通过 export MEFS_PATH=<local dir>的方式设置初始化的目录，然后再运行 init。

```shell
> mefs-keeper init --netKey=<net key> --sk=<your private key> --pwd=<your password> --keyfile=<absolute path of your keyfile>
```

参数解释：

- sk：私钥地址，不指定时默认新建一个账户；
- pwd：密码，不指定时为默认密码；
- keyfile: keyfile 文件的完整路径，不指定时默认为新建一个keyfile；
- netKey: 私有网络标志，现在有 dev 和 testnet；同一个标志的网络可以互联，默认为dev。

#### 修改网络传输端口

默认为 4001 端口，若需要改为<port num>，执行如下命令；
keeper 要求此 <port num> 可以被直接访问，或者在外网机器上有端口映射。

```shell
// 运行daemon前执行
mefs-keeper config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/<port num>\"]"
```

例如改为 4090 端口：

```shell
// 运行daemon前执行
mefs-keeper config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/4090\"]"
```

#### 启动实例

```shell
> mefs-keeper daemon --netKey= <net key> --pwd=<your password> --sk=<your secretKey>
```

参数解释：

- netKey: 私有网络标志，现在有 dev 和 testnet；同一个标志的网络可以互联，默认为dev；
- pwd：密码，不指定时为默认密码；
- sk：私钥地址，不指定时默认从本地私钥文件导出；

### 查看节点信息

通过`mefs-keeper id`可以查看节点的网络地址（8M...）、账户地址（0x...）、公钥、通信地址、版本信息。

```shell
mefs-keeper id <peerID> --pwd=<your password>
```

参数解释：

* peerID：待查寻的节点ID；默认为本地节点；
* pwd：该节点的密码，默认为DefaultPassword，代理节点则默认为空；


结果输出：

```shell
{
	"NetworkAddr": "8MJ5cAWfAP86cHmAcC3dxqzK41dh4a",
	"AccountAddr": "0x91b90d08aaE86F87648cF6B497918D487A5C19Ef",
	"PublicKey": "A6nefy7/FZbrrszX/itXdKmNTrFmkwMSYptFYwBjRVxz",
	"Addresses": [
		"/ip4/127.0.0.1/tcp/4001/p2p/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a",
		"/ip4/172.26.133.70/tcp/4001/p2p/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a",
		"/ip4/172.17.0.1/tcp/4001/p2p/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a"
	],
	"AgentVersion": "go-mefs/v0.3.2/ddf22678"
}
```

### 显示信息

使用`mefs-keeper info`用来显示与keeper节点相关的信息。

#### 显示收益

mefs-keeper通过提供管理服务获取收益。

```shell
mefs-keeper info list_income
```
输出结果为：

```shell
manageIncome: 88155981.17 Gwei
posIncome: 0 Wei
```
manageIncome代表通过管理服务获得的收益，posIncome代表通过冷启动管理服务获得的收益。

#### 显示其他节点地址

在一次存储服务中，keeper与user、provider签订存储服务合约，共同为user服务。通过下述命令可以查询keeper与哪些user签订了存储服务合约。

```shell
mefs-keeper info list_users
```

输出结果：

```shell
8MJacEyv1Pf92d9k93uquFzPcnG5rH.fsID:8MG8vi4NGKmdrz3EuxAgrPatAk5LJ1 has keepers:8MH4Woxb2FkM5nFr86dHj21fLgEybi/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a/8MK6qHvAfayLQy4d2684NTLAVLpQFk
8MJacEyv1Pf92d9k93uquFzPcnG5rH.fsID:8MG8vi4NGKmdrz3EuxAgrPatAk5LJ1 has providers:8MJJ58XHucLXfV5ej7GUXWhRDAHfim/8MGvZx9fireJe3McVXatDrim2CmNHJ/8MHdtmAtx5Q54So9MRWSSELorrZJdx/8MGXm9jEhY2JHAdpfbmbS4agqsJ2yM/8MKLYiSAFsdVxfj1PYNCQQjoG5Nwgm/8MH9yV7GZXhjYZeydVvZgEF3U7kxhN
```

此结果表明keeper节点与user节点（网络地址为8MJacEyv1Pf92d9k93uquFzPcnG5rH）签署了存储服务合约，8MG8vi4NGKmdrz3EuxAgrPatAk5LJ1表示文件系统ID，该ID代表了此次存储服务，由query合约地址转化而来。

查看与该keeper节点一起签订存储合约的其他keeper账户地址：

```shell
mefs-keeper info list_keepers
```

查看与该keeper节点一起签订存储合约的其他provider账户地址：

```shell
mefs-keeper info list_providers
```

### 查看网络节点

#### 查看网络中Keeper节点

展示在链上注册过的所有Keeper账户地址：

```shell
mefs-keeper list keepers
```

输出结果：

```shell
"KeeperCount": 3,
	"PledgeMoney": "30000000.00 Gwei", //Keeper质押总金额
	"OnlineCount": 2,//在线Keeper个数
	"OnlineKepepers": [ //在线的Keeper
		{
			"Address": "0x489799F06C6E70599e1Cc34d393394D59Cca1695",//账户地址
			"Online": true,
			"PledgeMoney": "10000000.00 Gwei",//质押金额
			"PledgeTime": "2020-09-28 Mon 13:01:29 CST"//质押时间
		},
		{
			"Address": "0xdB04F60f4313fdCb75C51679ad25dE5D5e92a45B",
			"Online": true,
			"PledgeMoney": "10000000.00 Gwei",
			"PledgeTime": "2020-09-28 Mon 13:05:10 CST"
		}
	],
	"OfflineCount": 1, //不在线的Keeper
	"OfflineKeepers": [
		{
			"Address": "0x91b90d08aaE86F87648cF6B497918D487A5C19Ef",
			"Online": false,
			"PledgeMoney": "10000000.00 Gwei",
			"PledgeTime": "2020-09-28 Mon 12:54:27 CST"
		}
	]
```
#### 查看网络中Provider节点

展示在链上注册过的所有Provider账户地址：

```shell
mefs-keeper list providers
```
