# MEFS 测试网使用

[TOC]

## 获取账号

// contact xxx

## 注册 provider

// visit xxx

## 设置主 keeper

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

## 启动 mefs

每个命令的参数解释见[使用文档](https://github.com/memoio/docs)

- mefs 初始化

```shell
> mefs init --netKey=testnet --sk=<your private key> --pwd=<your password>
```

- 启动 mefs 的网络实例，可后台运行

```shell
> mefs daemon --netKey=testnet --pwd=<your password> --dur=<your storage duration> --cap=<your storage capacity> --price=<your storage price> --rdo=false --pos=true
```

参数解释：

- dur: 提供的存储时间长度；按天计算， 默认是 365；
- cap：提供的存储空间大小；按 MB 计算，默认是 100000；
- price: 提供的存储价格，按 wei 计算，默认是 26500000000；
- rdo：是否重新部署 offer 合约，默认是 false。
- pos：是否使用冷启动功能，默认是 false，不开启；若使用，需要保证可用空间大于设置的存储空间
