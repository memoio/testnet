# MEFS 测试网使用

[TOC]

## 运行要求

docker 环境

## 获取账号

// contact xxx

输入密码，获得账号，导出 keyfile

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

## 进入终端

```shell
> sudo docker exec -it <container name> bash
```

- cli 方式使用见命令文档

- S3 接口使用见 S3 文档
