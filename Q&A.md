# Q&A

- 账户和私钥是什么？

```
mefs使用的私钥与账户和以太坊一样，格式为0x...；
```

- 角色是什么？

```
mefs包含3种角色，role，keeper，provider；目前每个账户地址对应一种角色，在启动的时候根据合约中的角色类型，启动不同的服务。
```

- 如何查看自己的角色？

```
在mefs daemon的启动过程中，可以查看输出的提示信息；也可以在运行过程中，运行`mefs test localinfo`查看自己的角色。
```

- 网页客户端可以做什么？

```
http://47.92.5.51，选择相应的角色入口，登录进入；可以进行质押，以及查看相应的信息。
```

- 如何查看自己的余额？

```
可以登录http://47.92.5.51/，查看自己的余额；也可以运行`mefs test showBalance`查看余额
```

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

- 如何设置区块链的 api 地址？

```
运行`mefs config Eth`，可以查看自己连接的区块链的地址，若想修改，可以运行`mefs config Eth xxx`, xxx为链的api地址。格式为`http://ip:port`
```

- 如何查看自己的网络连接状态？

```
运行`mefs swarm peers`查看自己连接的节点。
```
