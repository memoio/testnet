# Q&A

## common

- 账户和私钥是什么？

```
mefs使用的私钥与账户地址和以太坊一样，格式为0x...；
```

- 角色是什么？

```
mefs包含3种角色，role，keeper，provider；目前每个账户地址对应一种角色，在启动的时候根据合约中的角色类型，启动不同的服务。
```

- 账户地址和网络地址

```
启动mefs后会有两个地址：账户地址和网络地址；账户地址格式为0x...，网络地址格式为8M...；这两个地址实际上是等价的，一个用于和链交互，一个用于网络连接。
```

- 网页客户端可以做什么？

```
http://47.92.5.51，选择相应的角色入口，登录进入；可以进行质押，以及查看相应的信息。
```

- 如何查看自己的余额？

```
可以登录http://47.92.5.51/，查看自己的余额；也可以运行`mefs test showBalance`查看余额
```

- mefs 目录在哪里？包含哪些内容

```
`mefs init`会根据MEFS_PATH变量，在相应的目录下创建config，data，datastore，keystore等目录和文件。已经初始化的目录，再次运行`mefs init`会有报错信息进行提示；

如果想修改目录，在`mefs init`运行前设置MEFS_PATH即可；

keystore使用密码存储私钥；
data存储数据块的内容，用户的数据最终是以数据块的方式存放在data目录；
```

- 如看查看本地的账户地址？

```
运行`mefs id`可以看到本地的账户地址0x...
```

- 如看查看 mefs 的版本

```
运行`mefs id`可以看到mefs的版本号
```

- 如何查看自己的角色？

```
在mefs daemon的启动过程中，可以查看输出的提示信息；也可以在运行过程中，运行`mefs test localinfo`查看自己的角色。
```

## user

- user 的 LFS 功能是什么？

```
mefs 为user提供了一个加密的文件系统LFS，支持bucket和object操作
```

- user 的 LFS 如何启动？

```
在确认`mefs daemon`运行，而且运行的角色是user后，运行`mefs lfs start`；返回值返回时候，会有启动成功或者失败的信息，启动过程包括全网查询和合约签署，因而启动时间比较长。

user启动lfs的时候，可以选择使用默认参数；调低price参数，可能会找不到足够的provider；调高ks参数可能会找不到足够的keeper数量；调高ps参数，可能会找不到足够的provider。
```

- user 的 LFS 初始化错误有哪些？

  - 输入参数错误

  ```
  ps参数设置要大于1，其他参数要大于0；rdo为true/false
  ```

  - user 账户的金额不足

  ```
  设置的存储时长越大，存储大小越大，签署合约需要的金额越多；
  ```

  - 参数输入错误后，找不到足够的 keeper 数量/provider 数量怎么办？

  ```
  重新启动LFS，设置新的参数，rdo设置为true；
  ```

- LFS 的操作的常见错误有哪些？

  - 创建 bucket 的时候显示 bucket 已存在

  ```
  这说明已经用这个名字创建过bucket，现在bucket删除，只做标记记录，不是真正的删除，因此即使删除了bucket，再次使用此名字创建还是会显示bucket已存在。
  ```

  - 下载的时候，显示文件已存在

  ```
  这是由于当前目录已存在此文件，可以换一个目录下载，或者将当前目录下的文件重命名。
  ```

  - 上传的时候，显示文件已存在

  ```
  这说明当前上传的bucket中已有这个名字的文件，现在文件删除，只做标记记录，不是真正的删除，因此即使删除了文件，再次使用此名字创建还是会显示文件已存在。可以修改上传到的路径，例如`mefs lfs put_object objectName bucketName/<new dir name>`，上传到bucketName对应的<new dir name>的目录下。
  ```

## provider

- provider 的 pos 是什么？

```
pos功能是冷启动使用的，在provider刚加入网络的时候，存储的数据量较少，pos根据抵押的空间大小，生成本地数据，相应keeper挑战；在provider收到实际用户数据的时候，会逐步删除pso数据；pos数据和实际用户的数据区别在于：pos数据的价格为默认价格的1/10，pos数据不会被修复。
```

- provider 如何修改自己的价格？

```
provider在启动的时候，重新设置价格参数，将rdo设置为true，即可更新自己的存储价格。
```

- provider 如何设置自己的主 keeper？

```
provider 在运行的时候，可以通过`mefs contract addMasterKeeper 0x...`设置自己的主keeper；主keeper会优先提供自己的provider，以及触发upkeeping合约中的时空支付。
```

## others

- 系统中有哪些 keeper？

```
以下为公开的keeper的账户地址：
0x1adCa07Ae9bC70fc8c8d4C972176d1a1C810f0Ec
0xE434216FDF5573D8334Cb65cA2Df053e8A6f76C5
0x6Bd50cA3Ba83151f8Cb133B3C90737E173243adf
0xE561B5EAB2B97FAba9965eCC0179848D317ec2D3
0xf904237239a79f535bdc77622CCfB31E3B3f83C9
```

- 如何设置区块链的 api 地址？

```
运行`mefs config Eth`，可以查看自己连接的区块链的地址，若想修改，可以运行`mefs config Eth xxx`, xxx为链的api地址。格式为`http://ip:port`
```

- 如何查看自己的网络连接状态？

```
运行`mefs swarm peers`查看自己连接的节点。
```
