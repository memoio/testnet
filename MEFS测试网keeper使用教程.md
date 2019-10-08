# MEFS 测试网使用

[TOC]

## 运行要求

docker 环境，外网 ip，4001 端口开放

## 获取账号

// contact xxx

## 注册 keeper

// visit xxx

## 获取 mefs 镜像

将 docker 镜像 pull 下来

```shell
// to be releasing
> docker pull memoio/mefs-keeper
```

## 启动 docker

- 启动 docker

```shell
> docker run -itd -v <your local path>:/root/.mefs memoio/mefs-keeper
```

- 进入终端

```shell
> docker exec -it 9d304 bash
```

## 启动 mefs

每个命令的参数解释见[使用文档](https://github.com/memoio/docs)

- mefs 初始化

```shell
mefs init --netKey=testnet --sk=<your private key> --pwd=<your password>
```

- 启动 mefs 的实例，可后台运行

```shell
mefs daemon --netKey=testnet >> log 2>&1 &
```
