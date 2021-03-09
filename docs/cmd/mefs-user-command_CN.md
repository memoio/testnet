# MEFS 命令行操作文档

[TOC]

## 命令详情

### 启动

#### 初始化

mefs 初始化，默认初始化的目录为\$HOME/.mefs，可以通过 export MEFS_PATH=<local dir>的方式设置初始化的目录，然后再运行 init。

```shell
> mefs-user init --netKey=<net key> --sk=<your private key> --pwd=<your password> --keyfile=<absolute path of your keyfile>
```

参数解释：

- sk：私钥地址；
- pwd：密码；
- keyfile: keyfile 文件的完整路径；
- netKey: 私有网络标志，现在有 dev 和 testnet；同一个标志的网络可以互联。

#### 修改网络传输端口

默认为 4001 端口，若需要改为<port num>，执行如下命令；

```shell
// 运行daemon前执行
mefs-user config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/<port num>\"]"
```

例如改为 4090 端口：

```shell
// 运行daemon前执行
mefs-user config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/4090\"]"
```

#### 启动实例

```shell
// 以后台方式运行
> mefs-user daemon --netKey=<net key> --pwd=<your password> >> log 2>&1 &
```

参数解释：

- pwd：密码；
- netKey: 私有网络标志，现在有 dev 和 testnet

#### 启动用户 LFS

在启动 mefs 实例后，user 用户启动其存储空间。

启动期间由于需要匹配合约，部署合约，耗时约30分钟

```shell
> mefs-user lfs start <public address> --sk=<private key> --pwd=<password> --dur=<duration> --cap=<capacity> --price=<price> --ks=<keeper SLA> --ps=<provider SLA>
```

参数解释：

- public address：用户地址（0x...）；默认启动本地用户；
- sk：私钥地址； 默认从本地私钥文件导入；
- pwd：密码；默认使用默认密码；
- dur：提供的存储时间长度；按天计算， 默认是 100；
- cap：提供的存储大小；按 MB 计算，默认是 1TB；
- price：提供的存储价格，按 weiDollar/MB/hour 计算，默认是 4 * 10^9；约为 3 美元/(TB\*月)；
- ks：需要的 keeper 的数目，默认是 3；
- ps：需要的 provider 的数目，默认是 6；

### 使用 LFS（cli）

mefs 为每一个用户提供了一个专属的加密存储空间（LFS），每个存储空间包含多个桶（bucket），桶是用户用于存储对象（object）的容器，每个桶包含多个对象（object），我们可以把对象想象成文件。桶的冗余策略可以在创建的时候指定（存储在该桶中的所有对象都使用该种冗余策略），对象的数据使用对称加密方式加密。

user 可以通过命令行，网络（http），以及网关（gateway）的方式进行数据的操作。

也可以通过 sdk 进行操作，当前提供 go 版本。

#### 桶（bucket）操作

- 创建桶

命令描述：create_bucket 根据 BucketName 名字创建桶，每个桶可以设置不同的冗余策略，冗余策略为多副本（multiple replicas）或者纠删码（Reed-Solomon Codes）,可以调整数据块和校验块的个数来决定冗余水平。默认使用 3 个数据块和 2 个校验块的纠删码，可以容忍两个块的丢失。

```shell
> mefs-user lfs create_bucket <BucketName> --policy=<redundancy> --dc=<data count> --pc=<parity count> --addr=<public key>
```

参数解释：

```shell
BucketName: 桶的名字，最小3字节最大256字节；
--policy：冗余策略，--policy=1表示使用纠删码，--policy=2表示使用多副本，默认使用纠删码；
--dc：数据块的个数，默认是3；
--pc：校验块的个数，默认是2；当使用多副本策略的时候，实际的数据块为1，校验块为dc+pc-1
--addr: user的地址，默认为空，表示用户为本地节点地址。
```

结果输出：

```shell
Method: Create Bucket    // 命令名称
BucketName: <BucketName> // 创建的桶的名字
--BucketID: <BucketID>  // 桶的内部ID
--Ctime: <Ctime> // 创建时间
--Policy: <Policy> // 冗余策略
--DataCount: <DataCount> // 数据块个数
--ParityCount: <ParityCount>  // 校验块个数
```

- 桶列表

命令描述：list_buckets 显示出此用户创建的所有的桶，包含每个桶的名字(BucketName)，创建时间(Ctime)，冗余策略(Policy)和冗余参数(DataCount、ParityCount)

```shell
> mefs-user lfs list_buckets --addr=<public key>
```

参数解释：

```shell
--addr: user的地址，默认为空，表示用户为本地节点地址。
```

结果输出：

```shell
Method: List Buckets
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
--Policy: <Policy>
--DataCount: <DataCount>
--ParityCount: <ParityCount>
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
...
```

- 桶信息

命令描述：若 BucketName 名字的桶存在，head_bucket 显示此桶的创建时间，冗余策略和冗余参数；若不存在，返回桶不存在。

```shell
> mefs-user lfs head_bucket <BucketName> --addr=<public key>
```

参数解释：

```shell
BucketName：桶的名字
--addr: user的地址，默认为空，表示用户为本地节点地址
```

结果输出：

```shell
Method: Head Bucket
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
--Policy: <Policy>
--DataCount: <DataCount>
--ParityCount: <ParityCount>
```

- 删除桶（bucket）

命令描述：若 BucketName 名字的桶存在，delete_bucket 删除此桶；若不存在，返回桶不存在。只有在桶内为空的时候才会删除，否则返回桶不为空。

```shell
> mefs-user lfs delete_bucket <BucketName> --addr=<public key>
```

参数解释：

```shell
BucketName：桶的名字
--addr: user的地址，默认为空，表示用户为本地节点地址
```

结果输出：

```shell
Method: Delete Bucket
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
--Policy: <Policy>
--DataCount: <DataCount>
--ParityCount: <ParityCount>
```

#### 对象（object）操作

- 上传文件

命令描述：put_object 向 BucketName 桶内上传一个名为 ObjectName 的对象；若桶不存在，返回桶不存在；若对象已存在，则返回对象已存在。

```shell
> mefs-user lfs put_object <ObjectName> <BucketName> --addr=<public key>
```

参数解释：

```shell
ObjectName：上传的文件的名字，使用相对或者绝对路径；
BucketName：桶的名字；
--addr: user的地址，默认为空，表示用户为本地节点地址  // 为空时有异议
```

结果输出：

```shell
Method: Put Object
ObjectName: <ObjectName>  // 对象的名字
--ObjectSize: <ObjectSize> // 上传的对象的大小
--MD5: <MD5>               // 上传的对象经过MD5加密后得到的散列值
--Ctime: <Ctime>           // 此对象的创建时间
--Dir: <Dir>               // 是否是目录，true为目录，false为文件
--LatestChalTime:<LatestChalTime>  // 最近一次，此object被挑战的时间
```

- 下载文件

命令描述：get_object 从 BucketName 桶内下载一个名为 ObjectName 的对象；若桶不存在，返回桶不存在；若对象不存在，则返回对象不存在。

```shell
mefs-user lfs get_object <BucketName> <ObjectName> --o=<OutputName> --addr=<public key>
```

参数解释：

```shell
ObjectName：下载的对象名称；
BucketName：桶的名字；
--o: 默认下载对象的名字为ObjectName，若设置此参数，则为OutputName
--addr: user的地址，默认为空，表示用户为本地节点地址
```

结果输出：

下载的文件

- 文件列表

命令描述：list_objects 列出 BucketName 桶内所有的对象，包括对象大小，创建时间，MD5 值，是否是目录，最近被挑战的时间。

```shell
mefs-user lfs list_objects <BucketName> --addr=<public key>
```

参数解释：

```shell
BucketName：桶的名字；
--addr: user的地址，默认为空，表示用户为本地节点地址
```

结果输出：

```shell
Method: List Object
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--MD5: <MD5>
--Ctime: <Ctime>
--Dir: <Dir>
--LatestChalTime:<LatestChalTime>
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--MD5: <MD5>
--Ctime: <Ctime>
......
```

- 文件信息

命令描述：head_object 显示 BucketName 桶内 ObjectName 对象的大小，MD5 值，创建时间，是否为目录，最近被挑战的时间；

```shell
mefs-user lfs head_object <BucketName> <ObjectName> --addr=<public key>
```

参数解释：

```shell
ObjectName：下载的对象名称；
BucketName：桶的名字；
--addr: user的地址，默认为空，表示用户为本地节点地址
```

结果输出：

```shell
Method: Head Object
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--MD5: <MD5>
--Ctime: <Ctime>
--Dir: <Dir>
--LatestChalTime:<LatestChalTime>
```

- 删除文件

命令描述：delete_object 从 BucketName 桶内删除 ObjectName 对象。

```shell
 mefs-user lfs delete_object <BucketName> <ObjectName> --addr=<public key>
```

参数解释：

```shell
ObjectName：下载的对象名称；
BucketName：桶的名字；
--addr: user的地址，默认为空，表示用户为本地节点地址
```

结果输出：

```shell
Method: Delete Object
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--Ctime: <Ctime>
--Dir: <Dir>
--LatestChalTime:<LatestChalTime>
```

#### 角色操作

- 列出对应的 keeper

命令描述：list_keepers 列出与此 user 签署 UpKeeping 合约的 keeper

```shell
mefs-user lfs list_keepers
```

输出为对应的 keeper id

- 列出对应的 provider

命令描述：list_providers 列出给此 user 存储数据的 provider

```shell
mefs-user lfs list_providers
```

输出为对应的 provider id

#### 其他

- 刷新元数据

命令描述：fsync 手动刷新 lfs 的元数据，此命令在 keeper 上执行。元数据包括 SuperBlock、BucketInfo、ObjectInfo

```shell
mefs-user lfs fsync
```

输出为 Flush Success

- 查询使用的储存空间

命令描述：show_storage 查询用户的使用空间，即用户的所有 bucket 中共存储了多少数据，单位是 kb

```shell
mefs-user lfs show_storage --addr=<public key>
```

参数解释：

```shell
--addr: user的地址，默认为空，表示用户为本地节点地址
```

输出为相应的空间，格式为两位小数带单位（kb）

### http 操作

mefs 的命令都可以使用 http 进行操作

#### 配置

在 mefs-user 启动之前，进行如下配置：

```shell
// mefs api的端口设置,默认为5001
mefs-user config Addresses.API /ip4/0.0.0.0/tcp/5001
// 跨域访问
mefs-user config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]'
mefs-user config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST"]'
```

然后重新启动 mefs-user 即可使用 http 方式进行操作。

#### 使用

一个类似于如下的命令：

```shell
mefs-user rootcmd subcmd <arg1> <arg2> -opname1=<op1> -opname2=<op2>
```

对应的 http 请求为：

```shell
curl  "http://<ip>:<port>/api/v0/api/v0/<rootcmd>/<subcmd>?arg=<arg1>&arg=<arg2>&opname1=<op1>&opname2=<op2>"
```

ip 为启动 mefs-user 的机器的网络地址，port 默认为 5001，若运行前配置了跨域访问，可以使用外网 ip 进行访问，否则只能通过 127.0.0.1 访问。

#### example

- 显示所有 bucket 的信息

```shell
curl  "http://127.0.0.1:5001/api/v0/lfs/list_buckets?addr=<public key>"
```

输出是标准的 json 格式

```json
{
  "Method": "List Buckets",
  "Buckets": [
    {
      "BucketName": "<BucketName>",
      "BucketID": "<BucketID>",
      "Ctime": "<Ctime>",
      "Policy": "<Policy>",
      "DataCount": "<DataCount>",
      "ParityCount": "<ParityCount>"
    },
    {
      "BucketName": "<BucketName>",
      "BucketID": "<BucketID>",
      "Ctime": "<Ctime>",
      "Policy": "<Policy>",
      "DataCount": "<DataCount>",
      "ParityCount": "<ParityCount>"
    }
  ]
}
```

- 显示某 bucket 的所有 object 的信息

```shell
curl  "http://127.0.0.1:5001/api/v0/lfs/list_objects?arg=<BucketName>&addr=<public key>"
```

输出是标准的 json 格式

```json
{
  "Method": "List Objects",
  "Objects": [
    {
      "ObjectName": "<ObjectName>",
      "ObjectSize": "<ObjectSize>",
      "Ctime": "<Ctime>",
      "Dir": "<Dir>",
      "LatestChalTime": "<LatestChalTime>"
    },
    {
      "ObjectName": "<ObjectName>",
      "ObjectSize": "<ObjectSize>",
      "Ctime": "<Ctime>",
      "Dir": "<Dir>",
      "LatestChalTime": "<LatestChalTime>"
    }
  ]
}
```

### 网关模式

多个用户可以共用一个 mefs 的运行程序

#### user 代理启动

在启动 mefs 后，也可以代理启动其他的用户。

```shell
mefs-user lfs start <addr> --sk=<private key> --pwd=<password>
```

参数解释：

```shell
addr：用户地址;
--sk：用户的私钥;地址对应的私钥，若不匹配，以私钥的地址为准；
--pwd：用户密码;
```

#### user 代理关闭

在启动 mefs 后，也可以代理关闭其他的用户。

```go
mefs-user lfs kill addr --pwd=<password>
```

参数解释：

```shell
addr：用户地址
--pwd：用户密码
```

#### 使用

- cli

```shell
mefs-user rootcmd subcmd arg1 op1=arg2 --addr=<public key>
```

- http

```shell
curl  "http://<ip>:5001/api/v0/api/v0/<roocmd>/<subcmd>?arg=<arg1>&op1=<arg2>&addr=<public key>"
```

#### example

地址为 public key 的用户从 BucketName 桶内获取 ObjectName 名字的文件

- cli

```shell
mefs lfs get_object <BucketName> <ObjectName> --addr=<public key>
```

- http
```shell
curl  "http://127.0.0.1:5001/api/v0/lfs/get_object?arg=<BucketName>&arg=<ObjectName>&addr=<public key>"
```
