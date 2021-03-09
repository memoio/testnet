# MEFS 测试网使用

[TOC]

## 运行要求

* 推荐配置：4核，8G内存，1TB存储，20Mbps带宽；
* 外网ip，4001端口开放；
* docker环境；

## 获取Docker镜像

```
> docker pull memoio/mefs-provider:latest
```

## 生成账号

```
> docker run -it -v <your local storage dir>:/root --entrypoint="/app/create" memoio/mefs-provider:latest
```

之后按提示输入mefs-provider的密码（至少8位，默认是memoriae），mefs-provider的keyfile存放在 < your local storage >/.mefs/keystore目录下。

生成账号的例子如下：

```
> docker run -it -v ~/docker-test/provider:/root --entrypoint="/app/create" memoio/mefs-provider:latest
> please input your password(at least 8): 12345678
> Private Key: 5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894
  Address: 0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0
```

生成地址为"0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0"，私钥为”5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894“，keyfile存放在~/docker-test/provider/.mefs/keystore目录下，文件名为”0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0“，密码为”12345678“.

## 注册Provider

发送邮件至 sup@memolabs.io 申请provider注册

邮件内容：账户地址（比如上述生成的0x...）、角色（provider）

## 启动Provider

要求：机器的4001端口可以被公网访问，或者在出口机器上有端口映射

### 启动命令：

```
//启动docker；4001端口用于网络连接
sudo docker run -d --stop-timeout 30 \
    -p <External Port Num>:<Port Num> \
    -v <storage dir>:/root \
    -e TRANSPORT=<Port Num> \
    -e WALLET="0x..." \
    -e PASSWORD="<your password>" \
    -e STORAGESIZE="1TB" \
    -e STORAGEPRICE="4000000000"
    -e STORAGEDURATION="100"
    -e POSENABLE="false" \
    --mount type=bind,source="<keystore dir>",destination=/app/keystore \
    --name <countainer name> memoio/mefs-provider:latest
```

### 参数解释：

* TRANSPORT：< Port Num > 为程序的网络端口，默认为4001；< External Port Num > 为docker映射出去的端口，默认可以与< Port Num >相同；
* WALLET：用户地址（0x...），必须指明；
* PASSWORD：keyfile的密码，若是以docker后台方式运行provider，则必须指明；若是以前台方式运行，可以在运行过程中输入；
* STORAGESIZE：提供的存储空间大小，例如10GB、1000MB、1TB等，默认为1TB；
* STORAGEPRICE：提供存储空间单价，单位token/MB/hour，默认4 * 10^9；
* STORAGEDURATION：提供存储时间，单位day，默认100；
* POSENABLE：是否启用冷数据填充功能，设置true开启；默认为false；会填充质押数据量的 70\%;
* storage dir：数据目录；
* keystore dir：注册后导出的keyfile所在的位置，keyfile的名字即WALLET；

第一次启动需要大概20min左右。

### 日志文件：

在< storage dir >/.mefs 目录下，存在启动日志daemon.stdout.xx以及logs目录下的运行日志；

通过运行日志，可以查看provider节点运行时的状况；运行出错时，可以查看启动日志；

## 进入终端

```
// 进入docker
> sudo docker exec -it <container name> bash
```

每个命令的参数解释见[使用文档](/docs/cmd/mefs-provider-command_CN.md)

