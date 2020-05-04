# MEFS 测试网使用

[TOC]

## 运行要求

- 推荐配置：4 核，8G 内存，1TB 存储，20Mbps 带宽；
- 外网 ip，4001 端口开放；
- docker 环境；

## 获取账号

// contact xxx

输入密码，获得账号，导出 keyfile

## 注册 provider

// visit xxx

## 获取 mefs 镜像

将 docker 镜像 pull 下来

```shell
// 获取
> sudo docker pull memoio/mefs-provider:latest
```

## 启动

要求：机器的 4001 端口可以被公网访问，或者在出口机器上有端口映射

```docker
// 启动docker; 4001用于网络连接
sudo docker run -d --stop-timeout 30 \
    -p <External Port Num>:<Port Num> \
    -v <storage dir>:/root \
    -e TRANSPORT=<Port Num>
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
- STORAGESIZE：提供的存储空间大小，例如 10GB，1000MB，1TB 等；默认为 1TB；
- POSENABLE：是否启用冷数据填充功能，设置 true 开启；默认为 false；
- storage dir：数据目录；
- keystore dir：注册后导出的 keyfile 所在的位置，keyfile 的名字包含 <WALLET>；

日志文件：
<storage dir>/.mefs 下 启动日志 daemon.stdout.xx 以及 logs 目录内的运行日志；
在运行时，可以查看运行日志；运行出错的时候，可以查看启动日志。

## 进入终端

```shell
// 进入docker
> sudo docker exec -it <container name> bash
```

每个命令的参数解释见[使用文档](https://github.com/memoio/docs)
