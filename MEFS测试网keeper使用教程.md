# MEFS 测试网使用

[TOC]

## 获取账号

// contact xxx

## 注册keeper

// visit xxx

## 获取 mefs 镜像

将 docker 镜像 pull 下来

```shell
> docker pull memoio/mefs
```

## 启动 docker

- 启动 docker

```shell
> docker run -itd -v <your local path>:/root/.mefs memoio/mefs
```

- 进入终端

```shell
> docker exec -it 9d304 bash
```

## 启动mefs

每个命令的参数解释见[使用文档](https://github.com/memoio/docs)

- mefs 初始化
  
```shell
mefs init --sk=<your private key> --pwd=<your password> 
```

- 配置存储空间

配置的存储额度为数字+GB，例如100GB

```shell
mefs config Datastore.StorageMax 100GB
```

- 启动 mefs 的网络实例，可后台运行

```shell
mefs daemon --netKey=testnet --pos=true >> log 2>&1 &
```