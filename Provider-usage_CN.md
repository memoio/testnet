# MEFS 测试网使用

[TOC]

## 运行要求

- 推荐配置：4 核，8G 内存，1TB 存储，20Mbps 带宽；
- 外网 ip，4001 端口开放；
- docker 环境；

## 获取 mefs 镜像

将 docker 镜像 pull 下来

```shell
// 获取mefs-provider镜像
> docker pull memoio/mefs-provider:latest
```

## 获取账号

两种方法生成账号

+ 方法1

```shell
sudo docker run -it -v <your local storage dir>:/root --entrypoint="/app/create"  memoio/mefs-provider:latest

```

输入密码（至少 8 位， 默认密码memoriae）, keyfile 放在<your local storage dir>/.mefs/keystore目录下

例如：

```shell
docker run -it -v ~/docker-testa/provider:/root --entrypoint="/app/create"  memoio/mefs-provider:latest

Please input your password (at least 8): asdfghjk
Private Key: 5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894
Address: 0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0
```

生成的地址为"0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0", 私钥为”5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894“；keyfile放在~/docker-testa/provider/.mefs/keystore目录下，名字为"0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0"，密码为”asdfghjk“,。


+ 方法2

// contact xxx
输入密码（至少 8 位），获得账号，导出 keyfile；

## 注册 provider

// visit xxx； or send email to xxxx

1. 账号设置为 provider 角色
2. 若默认质押 1TB 空间，则需要转账给此账号 350 Token；


## 启动

要求：机器的 4001 端口可以被公网访问，或者在出口机器上有端口映射

```docker
// 启动docker; 4001用于网络连接
docker run -d --stop-timeout 30 \
    -p <External Port Num>:<Port Num> \
    -v <you local storage dir>:/root \
    -e TRANSPORT=<Port Num> \
    -e WALLET="0x..." \
    -e PASSWORD="<your password>" \
    -e STORAGESIZE="1TB" \
    -e POSENABLE="false" \
    --mount type=bind,source="<keystore dir>",destination=/app/keystore \
    --name <container name> memoio/mefs-provider:latest
```

参数解释：

- TRANSPORT：<Port Num>为程序的网络端口，默认为 4001；<External Port Num>为 docker 映射出去的端口，默认可以与<Port Num>相同；
- WALLET：用户地址（0x...）；require；
- PASSWORD: keyfile 的密码，若是以 docker 后台方式运行，必要；以前台方式运行，可以在运行过程中输入；
- STORAGESIZE：提供（质押）的存储空间大小，例如 10GB，1000MB，1TB 等；默认为 1TB；
- POSENABLE：是否启用冷数据填充功能，设置 true 开启；默认为 false；会填充质押数据量的70\%
- storage dir：数据目录；
- keystore dir：获取账号后的 keyfile 所在的位置（同一台机器上），keyfile 的名字包含 <WALLET>； 若使用方法1生成的地址， <you local storage dir>未改变，可不填；

日志文件：
<storage dir>/.mefs 下 启动日志 daemon.stdout.xx 以及 logs 目录内的运行日志；
在运行时，可以查看运行日志；运行出错的时候，可以查看启动日志和logs/error.log错误日志。


例如：

```
docker run -d --stop-timeout 30 -v ~/docker-testa/provider:/root  -e WALLET="0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0" -e PASSWORD="asdfghjk" --mount type=bind,source="/home/ubuntu/.mefs/keystore",destination=/app/keystore --name mefs-provider memoio/mefs-provider:latest
```
或

```
// 用方法1生成地址的时候
docker run -it --stop-timeout 30 -v ~/docker-testa/provider:/root -e WALLET="0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0" -e PASSWORD="asdfghjk" --name mefs-provider memoio/mefs-provider:latest
```


## 查看信息

```shell
// 进入docker
> docker exec -it <container name> bash
```

```shell
// docker内执行
mefs-provider info
```

可以看到类似如下的信息：

```
{
    // 账户地址
    "Address": "0x4F59650c8DC9974e6f51e681a7961B4Fc91C8159",
    // 账户余额
    "Balance": "1002.70 Token",
    // 质押的空间
    "PledgeBytes": "1.00 TiB",
    // 已使用空间
    "UsedBytes": "721.33 GiB",
    // 生成数据空间
    "PosBytes": "641.70 GiB",
    // 本地可用空间
    "LocalFreeBytes": "150.69 GiB",
    // Offer合约地址
    "OfferAddress": "0x0DB479927b032d5b98bBCA3858aA71e6b4cBcaeA",
    // 提供的存储量
    "OfferCapacity": "976.56 GiB",
    // 提供的存储价格
    "OfferPrice": "3.02 Dollar/(TiB*Month) (For now, 1 Dollar = 100 Token)",
    // 提供的存储时间长度
    "OfferDuration": "100 day",
    // 提供的存储开始时间
    "OfferStartTime": "2020-05-15 Fri 04:42:44 CST",
    // 总收入 
    "TotalIncome": "663329927.21 Gwei",
    // 存储有用数据的收入; 当前延迟3天支付
    "StorageIncome": "44853599.37 Gwei",
    // 读数据收入
    "DownloadIncome": "8684.39 Gwei",
    // 生成数据收入，当前延迟1-3天支付
    "PosIncome": "618467643.45 Gwei"
}
```

每个命令的参数解释见[使用文档](https://github.com/memoio/docs)
