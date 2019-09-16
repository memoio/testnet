# MEFS 测试网使用

[TOC]

## 获取账号

// contact xxx

## 获取 mefs 镜像

将 docker 镜像 pull 下来

```shell
> docker pull memoio/mefs
```

## 启动 docker

- 启动 docker

```shell
> sudo docker run -itd -v <your local path>:/root/.mefs -p 5001:5001 memoio/mefs
```

- 进入终端

```shell
> sudo docker exec -it 9d304 bash
```

- 运行 mefs lfs，检查是否安装成功

## 启动mefs

每个命令的参数解释见[使用文档](https://github.com/memoio/docs)

- mefs 初始化

```shell
mefs init --sk=<your private key> --pwd=<your password> 
```

- 启动 mefs 的网络实例，可后台运行

```shell
mefs daemon --netKey=testnet --pwd=<your password> >> log 2>&1 &
```

### 用户空间 lfs 的使用

- 启动用户

启动期间由于需要匹配合约，部署合约，耗时约 10~20 分钟

```shell
> mefs lfs start --pwd=<your password>
```

- 创建桶

可以在该用户的加密存储空间 lfs 中新建 bucket，policy=1 为纠删码，2 为多副本。

```shell
> mefs lfs create_bucket bucket01
> mefs lfs create_bucket bucket01 --policy=2
```

- 上传测试文件

```shell
> mefs lfs put_object /localpath/test.dat bucket01
```

- 下载文件，检验 md5 值是否正确

```shell
> mefs lfs get_object bucket02 test.dat
```

- 查看 lfs 空间的信息

```shell
> mefs lfs show_storage
```

## 3. HTTP-Client 使用

go 接口[mefs-http-api-go](https://github.com/memoio/mefs-http-api-go)
