# MEFS 测试网使用

[TOC]

## 运行要求

docker 环境

## 获取账号

// contact xxx

输入密码（至少 8 位），获得账号，导出 keyfile

## 获取 mefs 镜像

将 docker 镜像 pull 下来

```docker
// 获取
> sudo docker pull memoio/mefs-user:latest
```

## 启动

启动期间由于需要匹配合约，部署合约，耗时约 20~30 分钟

```shell
// 启动docker; 4001用于网络连接，5080用于S3接口
sudo docker run -d --stop-timeout 30 \
    -p 4001:4001 \
    -p 5080:5080 \
    -v <storage dir>:/root \
    -e WALLET="0x..." \
    -e PASSWORD="<your password>" \
    -e STORAGESIZE="1TB" \
    -e GATEWAY="true" \
    --mount type=bind,source="<keystore dir>",destination=/app/keystore \
    --name <container name> memoio/mefs-user:latest
```

参数解释：

- WALLET：用户地址（0x...）；require；
- PASSWORD: keyfile 的密码，若是以 docker 后台方式运行，必要；以前台方式运行，可以在运行过程中输入；
- STORAGESIZE：使用的存储空间大小，例如 10GB，1000MB，1TB 等；默认为 100GB；
- GATEWAY：是否开启 gateway 模式，开启后，5080 端口对外提供 minio S3 接口服务；用户名为 WALLET，密码为 PASSWORD；默认开启；
- storage dir：数据目录；
- keystore dir：注册后导出的 keyfile 所在的位置，keyfile 的名字包含 <WALLET>；

日志文件：
<storage dir>/.mefs 下 启动日志 daemon.stdout.xx 以及 logs 目录内的运行日志；
在运行时，可以查看运行日志；运行出错的时候，可以查看启动日志。

## 命令行操作文档

- 进入终端

```shell
> sudo docker exec -it <container name> bash
```

### 查看信息

```shell
> mefs-user lfs info
```

可以看到类似如下信息：

```
    {
        // 用户地址
        "UserAddr": "0xC2b27Aa18A1930D5b403b2021D8f52044C7B092B",
        // 用户余额
        "Balance": 1999996774493642600,
        // upkeeping合约的金额
        "UkBalance": 3996313779300347569,
        // 总空间 bytes
        "TotalBytes": 104857600,
        // 已使用空间 bytes
        "UsedBytes": 48441414,
        // 开始时间
        "StartTime": "2020-05-07 Thu 18:21:25 CST",
        // 结束时间
        "EndTime": "2294-02-20 Tue 18:21:25 CST",
        // 持续时间seconds
        "Duration": 8640000000,
        // 价格 wei/(MB*hour)
        "Price": 400000000,
        // upkeeping合约地址
        "UpKeepingAddr": "0x9B9fF75beC2Fff8cA2048315bc18c89e77826473",
        "QueryAddr": "0xD39af596b7F1452695955B9aC8dB00A7f15c79f8",
        // keeper的信息
        "KeeperAddrs": [
                "0x1adCa07Ae9bC70fc8c8d4C972176d1a1C810f0Ec",
                "0xE434216FDF5573D8334Cb65cA2Df053e8A6f76C5",
                "0xE561B5EAB2B97FAba9965eCC0179848D317ec2D3"
        ],
        // providers的信息
        "ProviderAddrs": [
                "0x2DE6e2fB47c7DE57D815A13A1FaC4eA62aa3fed8",
                "0x303466ccA8F7cc5f946507091AA962dd95CAA84A",
                "0x626235394f88ab0b5303D9638F04BC123728d201",
                "0xaBC8701a6Fd73877b54414CaaE97E3d2590a0bb7",
                "0xBB7adDBd37290d96b5db242B2fBdA03ED8c49983",
                "0xe2F6ED90EAF55B6A8Da297902C209A905f1d62E6"
        ]
}

```

### 使用 LFS（cli）

mefs 为每一个用户提供了一个专属的加密存储空间（LFS），每个存储空间包含多个桶（bucket），桶是用户用于存储对象（object）的容器，每个桶包含多个对象（object），我们可以把对象想象成文件。桶的冗余策略可以在创建的时候指定（存储在该桶中的所有对象都使用该种冗余策略），对象的数据使用对称加密方式加密。

user 可以通过命令行，浏览器，以及 sdk（golang 版本） 的方式进行数据的操作。

#### 桶（bucket）操作

- 创建桶

命令描述：create_bucket 根据 BucketName 名字创建桶，每个桶可以设置不同的冗余策略，冗余策略为多副本（multiple replicas）或者纠删码（Reed-Solomon Codes）,可以调整数据块和校验块的个数来决定冗余水平。默认使用 3 个数据块和 2 个校验块的纠删码，可以容忍两个块的丢失。

```shell
> mefs lfs create_bucket <BucketName> --policy=<redundancy> --dc=<data count> --pc=<parity count> --addr=<public key>
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
> mefs lfs list_buckets --addr=<public key>
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
> mefs lfs head_bucket <BucketName> --addr=<public key>
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
> mefs lfs delete_bucket <BucketName> --addr=<public key>
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
> mefs lfs put_object <ObjectName> <BucketName> --addr=<public key>
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
mefs lfs get_object <BucketName> <ObjectName> --o=<OutputName> --addr=<public key>
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
mefs lfs list_objects <BucketName> --addr=<public key>
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
mefs lfs head_object <BucketName> <ObjectName> --addr=<public key>
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
 mefs lfs delete_object <BucketName> <ObjectName> --addr=<public key>
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
mefs lfs list_keepers
```

输出为对应的 keeper id

- 列出对应的 provider

命令描述：list_providers 列出给此 user 存储数据的 provider

```shell
mefs lfs list_providers
```

输出为对应的 provider id

#### 其他

- 刷新元数据

命令描述：fsync 手动刷新 lfs 的元数据，此命令在 keeper 上执行。元数据包括 SuperBlock、BucketInfo、ObjectInfo

```shell
mefs lfs fsync
```

输出为 Flush Success

- 查询使用的储存空间

命令描述：show_storage 查询用户的使用空间，即用户的所有 bucket 中共存储了多少数据，单位是 kb

```shell
mefs lfs show_storage --addr=<public key>
```

参数解释：

```shell
--addr: user的地址，默认为空，表示用户为本地节点地址
```

输出为相应的空间，格式为两位小数带单位（B）

## 浏览器

若 GATEWAY 设置为 true，则可以浏览器访问本地的 5080 端口，网页使用；
用户名为 WALLET 值，密码为 PASSWORD 值；

## S3 接口

提供 minio S3 接口，暂未支持 ssl；用户名为 WALLET 值，密码为 PASSWORD 值（密码长度至少为 8 位）；

```go
package main

import (
    "fmt"

    "github.com/minio/minio-go"
)

func main() {
    endpoint := "127.0.0.1:5080"
    accessKeyID := "0x6A6AEB9840C8a42A9d8Ff55cf2c213F9b812ED0A"
    secretAccessKey := "memoriae"
    useSSL := false

    _, err := minio.New(endpoint, accessKeyID, secretAccessKey, useSSL)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("OK")
}
```

| 操作存储桶                        | 操作对象                                          |
| :-------------------------------- | :------------------------------------------------ |
| [`MakeBucket`](#MakeBucket)       | [`GetObject`](#GetObject)                         |
| [`ListBuckets`](#ListBuckets)     | [`PutObject`](#PutObject)                         |
| [`BucketExists`](#BucketExists)   |                                                   |
| [`RemoveBucket`](#RemoveBucket)   | [`StatObject`](#StatObject)                       |
| [`ListObjects`](#ListObjects)     | [`RemoveObject`](#RemoveObject)                   |
| [`ListObjectsV2`](#ListObjectsV2) | [`RemoveObjects`](#RemoveObjects)                 |
|                                   | [`FPutObject`](#FPutObject)                       |
|                                   | [`FGetObject`](#FGetObject)                       |
|                                   | [`PutObjectWithContext`](#PutObjectWithContext)   |
|                                   | [`GetObjectWithContext`](#GetObjectWithContext)   |
|                                   | [`FPutObjectWithContext`](#FPutObjectWithContext) |
|                                   | [`FGetObjectWithContext`](#FGetObjectWithContext) |

### 初始化连接

<a name="MinIO"></a>

- New(endpoint, accessKeyID, secretAccessKey string, ssl bool) (\*Client, error)
  初使化一个新的 client 对象。

**参数**

| 参数              | 类型     | 描述                |
| :---------------- | :------- | :------------------ |
| `endpoint`        | _string_ | MEFS 服务 endpoint  |
| `accessKeyID`     | _string_ | MEFS 账户的地址     |
| `secretAccessKey` | _string_ | MEFS 账户密码       |
| `ssl`             | _bool_   | true 代表使用 HTTPS |

### 2. 操作存储桶

<a name="MakeBucket"></a>

- MakeBucket(bucketName, location string) error

创建一个存储桶。

**参数**

| 参数         | 类型     | 描述         |
| ------------ | -------- | ------------ |
| `bucketName` | _string_ | 存储桶名称   |
| `location`   | _string_ | \*\*选项字段 |

**示例**

```go
err = minioClient.MakeBucket("mybucket", "")
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully created mybucket.")
```

<a name="ListBuckets"></a>

- ListBuckets() ([]BucketInfo, error)

列出所有的存储桶。

| 参数         | 类型                 | 描述                |
| ------------ | -------------------- | ------------------- |
| `bucketList` | _[]minio.BucketInfo_ | 所有存储桶的 list。 |

**minio.BucketInfo**

| 参数                  | 类型        | 描述             |
| --------------------- | ----------- | ---------------- |
| `bucket.Name`         | _string_    | 存储桶名称       |
| `bucket.CreationDate` | _time.Time_ | 存储桶的创建时间 |

**示例**

```go
buckets, err := minioClient.ListBuckets()
if err != nil {
    fmt.Println(err)
    return
}
for _, bucket := range buckets {
    fmt.Println(bucket)
}
```

<a name="BucketExists"></a>

- BucketExists(bucketName string) (found bool, err error)

检查存储桶是否存在。

**参数**

| 参数         | 类型     | 描述       |
| :----------- | :------- | :--------- |
| `bucketName` | _string_ | 存储桶名称 |

**返回值**

| 参数    | 类型    | 描述           |
| :------ | :------ | :------------- |
| `found` | _bool_  | 存储桶是否存在 |
| `err`   | _error_ | 标准 Error     |

**示例**

```go
found, err := minioClient.BucketExists("mybucket")
if err != nil {
    fmt.Println(err)
    return
}
if found {
    fmt.Println("Bucket found")
}
```

<a name="RemoveBucket"></a>

- RemoveBucket(bucketName string) error

删除一个存储桶，存储桶必须为空才能被成功删除。

**参数**

| 参数         | 类型     | 描述       |
| :----------- | :------- | :--------- |
| `bucketName` | _string_ | 存储桶名称 |

**示例**

```go
err = minioClient.RemoveBucket("mybucket")
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="ListObjects"></a>

- ListObjects(bucketName, prefix string, recursive bool, doneCh chan struct{}) <-chan ObjectInfo

列举存储桶里的对象。

**参数**

| 参数           | 类型            | 描述                                                                     |
| :------------- | :-------------- | :----------------------------------------------------------------------- |
| `bucketName`   | _string_        | 存储桶名称                                                               |
| `objectPrefix` | _string_        | 要列举的对象前缀                                                         |
| `recursive`    | _bool_          | `true`代表递归查找，`false`代表类似文件夹查找，以'/'分隔，不查子文件夹。 |
| `doneCh`       | _chan struct{}_ | 在该 channel 上结束 ListObjects iterator 的一个 message。                |

**返回值**

| 参数         | 类型                    | 描述                                              |
| :----------- | :---------------------- | :------------------------------------------------ |
| `objectInfo` | _chan minio.ObjectInfo_ | 存储桶中所有对象的 read channel，对象的格式如下： |

**minio.ObjectInfo**

| 属性                      | 类型        | 描述               |
| :------------------------ | :---------- | :----------------- |
| `objectInfo.Key`          | _string_    | 对象的名称         |
| `objectInfo.Size`         | _int64_     | 对象的大小         |
| `objectInfo.ETag`         | _string_    | 对象的 MD5 校验码  |
| `objectInfo.LastModified` | _time.Time_ | 对象的最后修改时间 |

```go
// Create a done channel to control 'ListObjects' go routine.
doneCh := make(chan struct{})

// Indicate to our routine to exit cleanly upon return.
defer close(doneCh)

isRecursive := true
objectCh := minioClient.ListObjects("mybucket", "myprefix", isRecursive, doneCh)
for object := range objectCh {
    if object.Err != nil {
        fmt.Println(object.Err)
        return
    }
    fmt.Println(object)
}
```

<a name="ListObjectsV2"></a>

- ListObjectsV2(bucketName, prefix string, recursive bool, doneCh chan struct{}) <-chan ObjectInfo

使用 listing API v2 版本列举存储桶中的对象。

**参数**

| 参数           | 类型            | 描述                                                                     |
| :------------- | :-------------- | :----------------------------------------------------------------------- |
| `bucketName`   | _string_        | 存储桶名称                                                               |
| `objectPrefix` | _string_        | 要列举的对象前缀                                                         |
| `recursive`    | _bool_          | `true`代表递归查找，`false`代表类似文件夹查找，以'/'分隔，不查子文件夹。 |
| `doneCh`       | _chan struct{}_ | 在该 channel 上结束 ListObjects iterator 的一个 message。                |

**返回值**

| 参数         | 类型                    | 描述                            |
| :----------- | :---------------------- | :------------------------------ |
| `objectInfo` | _chan minio.ObjectInfo_ | 存储桶中所有对象的 read channel |

```go
// Create a done channel to control 'ListObjectsV2' go routine.
doneCh := make(chan struct{})

// Indicate to our routine to exit cleanly upon return.
defer close(doneCh)

isRecursive := true
objectCh := minioClient.ListObjectsV2("mybucket", "myprefix", isRecursive, doneCh)
for object := range objectCh {
    if object.Err != nil {
        fmt.Println(object.Err)
        return
    }
    fmt.Println(object)
}
```

### 3. 操作对象

<a name="GetObject"></a>

- GetObject(bucketName, objectName string, opts GetObjectOptions) (\*Object, error)

返回对象数据的流，error 是读流时经常抛的那些错。

**参数**

| 参数         | 类型                     | 描述                                            |
| :----------- | :----------------------- | :---------------------------------------------- |
| `bucketName` | _string_                 | 存储桶名称                                      |
| `objectName` | _string_                 | 对象的名称                                      |
| `opts`       | _minio.GetObjectOptions_ | GET 请求的一些额外参数，像 encryption，If-Match |

**minio.GetObjectOptions**

| 参数             | 类型                | 描述                                                                                            |
| :--------------- | :------------------ | :---------------------------------------------------------------------------------------------- |
| `opts.Materials` | _encrypt.Materials_ | `encrypt`包提供的对流加密的接口，(更多信息，请看https://godoc.org/github.com/minio/minio-go/v6) |

**返回值**

| 参数     | 类型             | 描述                                                                                                    |
| :------- | :--------------- | :------------------------------------------------------------------------------------------------------ |
| `object` | _\*minio.Object_ | *minio.Object*代表了一个 object reader。它实现了 io.Reader, io.Seeker, io.ReaderAt and io.Closer 接口。 |

**示例**

```go
object, err := minioClient.GetObject("testbucket2", "myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
localFile, err := os.Create("/tmp/local-file.jpg")
if err != nil {
    fmt.Println(err)
    return
}
if _, err = io.Copy(localFile, object); err != nil {
    fmt.Println(err)
    return
}
```

<a name="FGetObject"></a>

- FGetObject(bucketName, objectName, filePath string, opts GetObjectOptions) error

下载并将文件保存到本地文件系统。

**参数**

| 参数         | 类型                     | 描述                                            |
| :----------- | :----------------------- | :---------------------------------------------- |
| `bucketName` | _string_                 | 存储桶名称                                      |
| `objectName` | _string_                 | 对象的名称                                      |
| `filePath`   | _string_                 | 下载后保存的路径                                |
| `opts`       | _minio.GetObjectOptions_ | GET 请求的一些额外参数，像 encryption，If-Match |

**示例**

```go
err = minioClient.FGetObject("mybucket", "myobject", "/tmp/myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="GetObjectWithContext"></a>

- GetObjectWithContext(ctx context.Context, bucketName, objectName string, opts GetObjectOptions) (\*Object, error)

和 GetObject 操作是一样的，不过传入了取消请求的 context。

**参数**

| 参数         | 类型                     | 描述                                            |
| :----------- | :----------------------- | :---------------------------------------------- |
| `ctx`        | _context.Context_        | 请求上下文（Request context）                   |
| `bucketName` | _string_                 | 存储桶名称                                      |
| `objectName` | _string_                 | 对象的名称                                      |
| `opts`       | _minio.GetObjectOptions_ | GET 请求的一些额外参数，像 encryption，If-Match |

**返回值**

| 参数     | 类型             | 描述                                                                                                    |
| :------- | :--------------- | :------------------------------------------------------------------------------------------------------ |
| `object` | _\*minio.Object_ | *minio.Object*代表了一个 object reader。它实现了 io.Reader, io.Seeker, io.ReaderAt and io.Closer 接口。 |

**示例**

```go
ctx, cancel := context.WithTimeout(context.Background(), 100 * time.Second)
defer cancel()

object, err := minioClient.GetObjectWithContext(ctx, "mybucket", "myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}

localFile, err := os.Create("/tmp/local-file.jpg")
if err != nil {
    fmt.Println(err)
    return
}

if _, err = io.Copy(localFile, object); err != nil {
    fmt.Println(err)
    return
}
```

<a name="FGetObjectWithContext"></a>

- FGetObjectWithContext(ctx context.Context, bucketName, objectName, filePath string, opts GetObjectOptions) error

和 FGetObject 操作是一样的，不过允许取消请求。

**参数**

| 参数         | 类型                     | 描述                                            |
| :----------- | :----------------------- | :---------------------------------------------- |
| `ctx`        | _context.Context_        | 请求上下文                                      |
| `bucketName` | _string_                 | 存储桶名称                                      |
| `objectName` | _string_                 | 对象的名称                                      |
| `filePath`   | _string_                 | 下载后保存的路径                                |
| `opts`       | _minio.GetObjectOptions_ | GET 请求的一些额外参数，像 encryption，If-Match |

**示例**

```go
ctx, cancel := context.WithTimeout(context.Background(), 100 * time.Second)
defer cancel()

err = minioClient.FGetObjectWithContext(ctx, "mybucket", "myobject", "/tmp/myobject", minio.GetObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="PutObject"></a>

- PutObject(bucketName, objectName string, reader io.Reader, objectSize int64,opts PutObjectOptions) (n int, err error)

当对象小于 128MiB 时，直接在一次 PUT 请求里进行上传。当大于 128MiB 时，根据文件的实际大小，PutObject 会自动地将对象进行拆分成 128MiB 一块或更大一些进行上传。对象的最大大小是 5TB。

**参数**

| 参数         | 类型                     | 描述                                                                             |
| :----------- | :----------------------- | :------------------------------------------------------------------------------- |
| `bucketName` | _string_                 | 存储桶名称                                                                       |
| `objectName` | _string_                 | 对象的名称                                                                       |
| `reader`     | _io.Reader_              | 任意实现了 io.Reader 的 GO 类型                                                  |
| `objectSize` | _int64_                  | 上传的对象的大小，-1 代表未知。                                                  |
| `opts`       | _minio.PutObjectOptions_ | 允许用户设置可选的自定义元数据，内容标题，加密密钥和用于分段上传操作的线程数量。 |

**minio.PutObjectOptions**

| 属性                           | 类型                    | 描述                                                                                            |
| :----------------------------- | :---------------------- | :---------------------------------------------------------------------------------------------- |
| `opts.UserMetadata`            | _map[string]string_     | 用户元数据的 Map                                                                                |
| `opts.Progress`                | _io.Reader_             | 获取上传进度的 Reader                                                                           |
| `opts.ContentType`             | _string_                | 对象的 Content type， 例如"application/text"                                                    |
| `opts.ContentEncoding`         | _string_                | 对象的 Content encoding，例如"gzip"                                                             |
| `opts.ContentDisposition`      | _string_                | 对象的 Content disposition, "inline"                                                            |
| `opts.ContentLanguage`         | _string_                | 对象的 Content Language, "French"                                                               |
| `opts.CacheControl`            | _string_                | 指定针对请求和响应的缓存机制，例如"max-age=600"                                                 |
| `opts.Mode`                    | _\*minio.RetentionMode_ | 设置保留模式, e.g "COMPLIANCE"                                                                  |
| `opts.RetainUntilDate`         | _\*time.Time_           | 保留模式的有效期                                                                                |
| `opts.ServerSideEncryption`    | _encrypt.ServerSide_    | `encrypt`包提供的对流加密的接口，(更多信息，请看https://godoc.org/github.com/minio/minio-go/v6) |
| `opts.StorageClass`            | _string_                | 指定对象的存储类 MinIO 服务器支持的值有 `REDUCED_REDUNDANCY` 和 `STANDARD`                      |
| `opts.WebsiteRedirectLocation` | _string_                | 指定对象重定向到同一个桶的其他的对象或者外部的 URL                                              |

**示例**

```go
file, err := os.Open("my-testfile")
if err != nil {
    fmt.Println(err)
    return
}
defer file.Close()

fileStat, err := file.Stat()
if err != nil {
    fmt.Println(err)
    return
}

n, err := minioClient.PutObject("mybucket", "myobject", file, fileStat.Size(), minio.PutObjectOptions{ContentType:"application/octet-stream"})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

API 方法在 minio-go SDK 版本 v3.0.3 中提供的 PutObjectWithSize，PutObjectWithMetadata，PutObjectStreaming 和 PutObjectWithProgress 被替换为接受指向 PutObjectOptions struct 的指针的新的 PutObject 调用变体。

<a name="PutObjectWithContext"></a>

- PutObjectWithContext(ctx context.Context, bucketName, objectName string, reader io.Reader, objectSize int64, opts PutObjectOptions) (n int, err error)

和 PutObject 是一样的，不过允许取消请求。

**参数**

| 参数         | 类型                     | 描述                                                                                                                                                                                |
| :----------- | :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ctx`        | _context.Context_        | 请求上下文                                                                                                                                                                          |
| `bucketName` | _string_                 | 存储桶名称                                                                                                                                                                          |
| `objectName` | _string_                 | 对象的名称                                                                                                                                                                          |
| `reader`     | _io.Reader_              | 任何实现 io.Reader 的 Go 类型                                                                                                                                                       |
| `objectSize` | _int64_                  | 上传的对象的大小，-1 代表未知                                                                                                                                                       |
| `opts`       | _minio.PutObjectOptions_ | 允许用户设置可选的自定义元数据，content-type，content-encoding，content-disposition 以及 cache-control headers，传递加密模块以加密对象，并可选地设置 multipart put 操作的线程数量。 |

**示例**

```go
ctx, cancel := context.WithTimeout(context.Background(), 10 * time.Second)
defer cancel()

file, err := os.Open("my-testfile")
if err != nil {
    fmt.Println(err)
    return
}
defer file.Close()

fileStat, err := file.Stat()
if err != nil {
    fmt.Println(err)
    return
}

n, err := minioClient.PutObjectWithContext(ctx, "my-bucketname", "my-objectname", file, fileStat.Size(), minio.PutObjectOptions{
	ContentType: "application/octet-stream",
})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

<a name="CopyObject"></a>

- FPutObject(bucketName, objectName, filePath, opts PutObjectOptions) (length int64, err error)

将 filePath 对应的文件内容上传到一个对象中。

当对象小于 128MiB 时，FPutObject 直接在一次 PUT 请求里进行上传。当大于 128MiB 时，根据文件的实际大小，FPutObject 会自动地将对象进行拆分成 128MiB 一块或更大一些进行上传。对象的最大大小是 5TB。

**参数**

| 参数         | 类型                     | 描述                                                                                                                                                                                |
| :----------- | :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `bucketName` | _string_                 | 存储桶名称                                                                                                                                                                          |
| `objectName` | _string_                 | 对象的名称                                                                                                                                                                          |
| `filePath`   | _string_                 | 要上传的文件的路径                                                                                                                                                                  |
| `opts`       | _minio.PutObjectOptions_ | 允许用户设置可选的自定义元数据，content-type，content-encoding，content-disposition 以及 cache-control headers，传递加密模块以加密对象，并可选地设置 multipart put 操作的线程数量。 |

**示例**

```go
n, err := minioClient.FPutObject("my-bucketname", "my-objectname", "my-filename.csv", minio.PutObjectOptions{
	ContentType: "application/csv",
});
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

<a name="FPutObjectWithContext"></a>

- FPutObjectWithContext(ctx context.Context, bucketName, objectName, filePath, opts PutObjectOptions) (length int64, err error)

和 FPutObject 操作是一样的，不过允许取消请求。

**参数**

| 参数         | 类型                     | 描述                                                                                                                                                                                |
| :----------- | :----------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ctx`        | _context.Context_        | 请求上下文                                                                                                                                                                          |
| `bucketName` | _string_                 | 存储桶名称                                                                                                                                                                          |
| `objectName` | _string_                 | 对象的名称                                                                                                                                                                          |
| `filePath`   | _string_                 | 要上传的文件的路径                                                                                                                                                                  |
| `opts`       | _minio.PutObjectOptions_ | 允许用户设置可选的自定义元数据，content-type，content-encoding，content-disposition 以及 cache-control headers，传递加密模块以加密对象，并可选地设置 multipart put 操作的线程数量。 |

**示例**

```go
ctx, cancel := context.WithTimeout(context.Background(), 100 * time.Second)
defer cancel()

n, err := minioClient.FPutObjectWithContext(ctx, "mybucket", "myobject.csv", "/tmp/otherobject.csv", minio.PutObjectOptions{ContentType:"application/csv"})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println("Successfully uploaded bytes: ", n)
```

<a name="StatObject"></a>

- StatObject(bucketName, objectName string, opts StatObjectOptions) (ObjectInfo, error)
  获取对象的元数据。

**参数**

| 参数         | 类型                      | 描述                                                      |
| :----------- | :------------------------ | :-------------------------------------------------------- |
| `bucketName` | _string_                  | 存储桶名称                                                |
| `objectName` | _string_                  | 对象的名称                                                |
| `opts`       | _minio.StatObjectOptions_ | GET info/stat 请求的一些额外参数，像 encryption，If-Match |

**返回值**

| 参数      | 类型               | 描述           |
| :-------- | :----------------- | :------------- |
| `objInfo` | _minio.ObjectInfo_ | 对象 stat 信息 |

**minio.ObjectInfo**

| 属性                   | 类型        | 描述                |
| :--------------------- | :---------- | :------------------ |
| `objInfo.LastModified` | _time.Time_ | 对象的最后修改时间  |
| `objInfo.ETag`         | _string_    | 对象的 MD5 校验码   |
| `objInfo.ContentType`  | _string_    | 对象的 Content type |
| `objInfo.Size`         | _int64_     | 对象的大小          |

**示例**

```go
objInfo, err := minioClient.StatObject("mybucket", "myobject", minio.StatObjectOptions{})
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(objInfo)
```

<a name="RemoveObject"></a>

### RemoveObject(bucketName, objectName string) error

删除一个对象。

**参数**

| 参数         | 类型     | 描述       |
| :----------- | :------- | :--------- |
| `bucketName` | _string_ | 存储桶名称 |
| `objectName` | _string_ | 对象的名称 |

```go
err = minioClient.RemoveObject("mybucket", "myobject")
if err != nil {
    fmt.Println(err)
    return
}
```

<a name="RemoveObjects"></a>

- RemoveObjects(bucketName string, objectsCh chan string) (errorCh <-chan RemoveObjectError)

从一个 input channel 里删除一个对象集合。一次发送到服务端的删除请求最多可删除 1000 个对象。通过 error channel 返回的错误信息。

**参数**

| 参数         | 类型          | 描述                   |
| :----------- | :------------ | :--------------------- |
| `bucketName` | _string_      | 存储桶名称             |
| `objectsCh`  | _chan string_ | 要删除的对象的 channel |

**返回值**

| 参数      | 类型                             | 描述                                        |
| :-------- | :------------------------------- | :------------------------------------------ |
| `errorCh` | _<-chan minio.RemoveObjectError_ | 删除时观察到的错误的 Receive-only channel。 |

```go
objectsCh := make(chan string)

// Send object names that are needed to be removed to objectsCh
go func() {
	defer close(objectsCh)
	// List all objects from a bucket-name with a matching prefix.
	for object := range minioClient.ListObjects("my-bucketname", "my-prefixname", true, nil) {
		if object.Err != nil {
			log.Fatalln(object.Err)
		}
		objectsCh <- object.Key
	}
}()

for rErr := range minioClient.RemoveObjects("mybucket", objectsCh) {
    fmt.Println("Error detected during deletion: ", rErr)
}
```
