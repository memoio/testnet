# MEFS 测试网使用

[TOC]

## 运行要求

* 推荐配置：4核，8G内存，20Mbps带宽；
* 外网ip，4001端口开放；
* docker环境；

## 获取Docker镜像

```shell
> docker pull memoio/mefs-user:latest
```

## 生成账号

```shell
> docker run -it -v <your local storage dir>:/root --entrypoint="/app/create" memoio/mefs-user:latest
```

之后按提示输入mefs-user的密码（至少8位，默认是memoriae），mefs-user的keyfile存放在 < your local storage >/.mefs/keystore目录下。

生成账号的例子如下：

```shell
> docker run -it -v ~/docker-test/user:/root --entrypoint="/app/create" memoio/mefs-user:latest
> please input your password(at least 8): 12345678
> Private Key: 5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894
  Address: 0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0
```

生成地址为"0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0"，私钥为”5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894“，keyfile存放在~/docker-test/user/.mefs/keystore目录下，文件名为”0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0“，密码为”12345678“.

## 申请测试代币

发送邮件至 sup@memolabs.io 申请user测试代币

邮件内容：账户地址（比如上述生成的0x...）、角色（user）

## 启动

初次启动期间由于需要部署合约、匹配节点，耗时约 30 分钟

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

### 参数解释：

- WALLET：用户地址（0x...），必须指明；
- PASSWORD: keyfile 的密码，若是以 docker 后台方式运行，必要；以前台方式运行，可以在运行过程中输入；
- STORAGESIZE：使用的存储空间大小，例如 10GB，1000MB，1TB 等；默认为 1TB；
- GATEWAY：是否开启 gateway 模式，开启后，5080 端口对外提供 minio S3 接口服务；用户名为 WALLET，密码为 PASSWORD；默认开启；
- storage dir：数据目录；
- keystore dir：注册后导出的 keyfile 所在的位置，keyfile 的名字即 WALLET；

### 日志文件：
在< storage dir >/.mefs 目录下，启动日志 daemon.stdout.xx 以及 logs 目录内的运行日志；
在运行时，可以查看运行日志；运行出错的时候，可以查看启动日志。

## 进入终端

```shell
> sudo docker exec -it <container name> bash
```

- cli 方式使用见[命令文档](https://github.com/memoio/docs/cmd)

- S3 接口使用见[S3文档](https://github.com/memoio/docs/api)