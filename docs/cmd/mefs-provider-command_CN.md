# MEFS 命令行操作文档

[TOC]

## 命令详情

### 启动

#### 初始化

mefs 初始化，默认初始化的目录为\$HOME/.mefs，可以通过 export MEFS_PATH=<local dir>的方式设置初始化的目录，然后再运行 init。

```shell
> mefs-provider init --netKey=<net key> --sk=<your private key> --pwd=<your password> --keyfile=<absolute path of your keyfile>
```

参数解释：

- sk：私钥地址；
- pwd：密码；
- keyfile: keyfile 文件的完整路径；
- netKey: 私有网络标志，现在有 dev 和 testnet；同一个标志的网络可以互联。

#### 修改网络传输端口

默认为 4001 端口，若需要改为<port num>，执行如下命令；
provider 要求此 <port num> 可以被直接访问，或者在外网机器上有端口映射。

```shell
// 运行daemon前执行
mefs-provider config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/<port num>\"]"
```

例如改为 4090 端口：

```shell
// 运行daemon前执行
mefs-provider config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/4090\"]"
```

#### 启动实例

```shell
> mefs-provider daemon --netKey= <net key> --pwd=<your password> --dur=<your storage duration> --cap=<your storage capacity> --price=<your storage price> --deCap=<your deposit capacity> --rdo=<bool> --pos=true
```

参数解释：

- dur: 提供的存储时间长度；按天计算， 默认是 365 天；
- cap：提供的存储空间大小；按 MB 计算，默认是 1TB；
- price: 提供的存储价格，按 wei 计算，默认是 4 * 10^9 wei；即 3 美元/(TB\*月)；
- rdo：是否重新部署 offer 合约，默认是 false；
- deCap：质押的空间；按 MB 计算，默认 1TB；
- pos：是否使用冷启动功能，默认是 false，不开启；若使用，需要保证可用空间大于质押的空间；
- netKey: 私有网络标志，现在有 dev 和 testnet；同一个标志的网络可以互联；

### 查看节点信息

通过`mefs-provider id`可以查看节点的网络地址（8M...）、账户地址（0x...）、公钥、通信地址、版本信息。

```shell
mefs-provider id <peerID> --pwd=<your password>
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

### 展示信息

#### 本地节点信息

```shell
mefs-provider info self
```

输出结果：

```shell
"Wallet": "0xa12626B388dc08481520B0f9c7009b9505f87f5e", //账户地址
	"StartTime": "2020-11-16 Mon 10:24:50 CST",//注册时间
	"UpTime": "45 day 7 hour 42 minute 10 second",//上线时长
	"ReadyForService": true,//service服务是否已启动
	"PublicNetwork": "/ip4/58.210.46.6/tcp/4115/p2p/8MJJ58XHucLXfV5ej7GUXWhRDAHfim",//公网地址
	"PublicReachable": true,
	"Balance": "10874.62 Token",
	"PledgeBytes": "1.00 TiB",
	"UsedBytes": "639.31 GiB",
	"PosBytes": "0 B",
	"LocalFreeBytes": "232.77 GiB",
	"OfferAddress": "0x8bc53304465Aa7214Baaf4407f6d4626E15bBab0",
	"OfferCapacity": "976.56 GiB",
	"OfferPrice": "3.02 Dollar/(TiB*Month) (For now, 1 Dollar = 100 Token)",
	"OfferDuration": "8640000 day",
	"OfferStartTime": "2020-05-15 Fri 04:38:38 CST",
	"TotalIncome": "2.14 Token",
	"StorageIncome": "1.51 Token",
	"DownloadIncome": "7498443.30 Gwei",
	"PosIncome": "624882551.42 Gwei"
```

#### user信息

```shell
mefs-provider info users
```

输出结果：

```shell
8MJg6VvVqYPFr2d38wxUVGRZqqJcnv/8MGXdiJq3AhHuQa3dh1pucP6rJXahb
8MKWbYg3b9Gxj2c7szWkzSxAJS3V6y/8MHLoGHF2rBXPocBztUu2tAEaiqYqb
8MJr4bwoDcsGVdD7hhAJYMs8FkN3Kt/8MKACLbaSreLC2e7nkfEjPZvbeLhzk
8MKDyauiihGANvuAxc5G3N6B922rFy/8MKbFLenGA3zbNmKiZvkWaoztGLvJu
```
第一个ID表示Provider节点与之签订存储服务合约的User，第一个表示文件系统ID。

#### group信息

Provider为User提供存储服务，从而组成一个group，group信息包含存储服务的详细信息。

```shell
mefs-provider info group <uid> <qid>
```

参数解释：

* uid：user的ID；
* qid：文件系统的ID;

### 查看网络节点

#### 查看网络中Keeper节点

展示在链上注册过的所有Keeper账户地址：

```shell
mefs-provider list keepers
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
mefs-provider list providers
```