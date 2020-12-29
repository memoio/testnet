# Q&A

## key node information

### view provider startup information

1. daemon.stdout.xxx log

```log
Initializing daemon...  /* Begin to start, if you don’t have this step, the account or password is wrong*/
go-mefs version: v0.3.2-7794494f    /* mefs version */
Repo version: 1
System version: amd64/linux
Golang version: go1.13.4
Successfully raised file descriptor limit to 2048.
Using private network:  testnet                    /*Surface network type*/
Swarm is limited to private network of peers with the swarm key
Swarm key fingerprint: 2e4bbb87e7f8cbca64f6505632aa8f92
Swarm listening on /ip4/127.0.0.1/tcp/4001         /*Listening port*/
Swarm listening on /ip4/172.17.0.1/tcp/4001
Swarm listening on /ip4/172.21.10.101/tcp/4001
Swarm listening on /p2p-circuit
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip4/172.17.0.1/tcp/4001
Swarm announcing /ip4/172.21.10.101/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
Network daemon is ready   /*The network process is started successfully. If you cannot reach this step, the port is not available and you need to change the port*/
Starting as a provider    /*The provider process starts and goes to the running log*/
```

2. provider run log: logs/info.log

```log
{"level":"INFO","time":"2020-05-25T16:02:20+08:00","caller":"utils/log.go:71","msg":"Mefs Logger init success"}  //Log initialization succeeded
{"level":"INFO","time":"2020-05-25T16:02:23+08:00","caller":"mefs-provider/daemon.go:362","msg":"Init BLS12_381 curve success"}  //BLS12_381 initialized successfully
{"level":"INFO","time":"2020-05-25T16:02:23+08:00","caller":"role/contracts.go:293","msg":"Begin to deploy offer contract..."}    // begin deploy contract
{"level":"INFO","time":"2020-05-25T16:02:23+08:00","caller":"role/contracts.go:305","msg":"8MGpV2fUoVBZmHVgbC6BffQA1WL3EW (0x373a6F73387337873F8aFc90a7fd0e6Fa44fD4dF) has balance: 296635706423385865506"} /*balance,contract deployment will fail when the balance is insufficient；8MGpV2fUoVBZmHVgbC6BffQA1WL3EW is net address,0x373a6F73387337873F8aFc90a7fd0e6Fa44fD4dF is wallet address*/
{"level":"INFO","time":"2020-05-25T16:02:23+08:00","caller":"role/contracts.go:317","msg":"Finish deploy offer contract: 8MGtBopZAF1UPMXXZaex9KtDphnftX"}  // offer-contract deployed successfully
{"level":"INFO","time":"2020-05-25T16:02:24+08:00","caller":"provider/provider.go:186","msg":"Get 8MGpV2fUoVBZmHVgbC6BffQA1WL3EW's contract info success"}
{"level":"INFO","time":"2020-05-25T16:02:24+08:00","caller":"provider/provider.go:208","msg":"Provider Service is ready"}  /*provider service start successfully;Failure to reach this step indicates that the contract deployment is unsuccessful, usually due to insufficient balance*/
{"level":"INFO","time":"2020-05-25T16:02:24+08:00","caller":"provider/provider.go:815","msg":"Get infos from chain start!"}  // Query information from the chain
{"level":"INFO","time":"2020-05-25T16:02:24+08:00","caller":"provider/provider.go:849","msg":"Send storages to keepers start!"}  // Synchronize information to keeper
{"level":"INFO","time":"2020-05-25T16:02:24+08:00","caller":"provider/pos.go:55","msg":"Start Pos Service"}  // If pos is set true, start pos service and generate data
```

## run

#### "Illegal instruction (core dumped)" appears when running mefs in Docker

> Check the host's information `dmesg -c`, if a sentence like "traps: mefs[19703] trap invalid opcode ip:7fcfbb17a1b5 sp:7ffd2d73b210 error:0 in libmclbn384.so[7fcfbb165000+39000" appears, this error is caused by dynamic library references. Then the mcl library needs to be recompiled, just run /app/check_mcl in docker.

## common

#### What are the account and private key?

> The private key and account address used by mefs is the same as the Ethereum, and the format is 0x...;

#### What is the role?

> Mefs contains 3 roles: user, keeper, and provider; currently each account address corresponds to a role;
> During startup, different services are started according to the role type in the contract.

#### Account address and network address

> After starting mefs, there will be two addresses: account address and network address; the account address format is 0x..., the network address format is 8M...;
> These two addresses are actually equivalent, one for interacting with the chain, and one for network connection.

#### How to check my balance?

> Run `mefs-user/keeper/provider test showBalance` in the command line to view the balance of the running account, the unit is wei;

#### Where is the mefs directory? What's included?

> `mefs-user/keeper/provider init` will create config, data, datastore, keystore and other directories and files in the corresponding directories according to the MEFS_PATH variable;
>
> For the directory that has been initialized, if you run `mefs-user/keeper/provider init` again, an error message will prompt you;
>
> If you want to modify the directory, set MEFS_PATH before running `mefs-user/keeper/provider init`;
>
> The Keystore uses a password to store the private key;
> Data directory stores the content of the data block, and the user's data is finally stored in the Data directory in the form of data blocks;

#### How to check the local account address?

> Run `mefs-user/keeper/provider id` to see the local account address 0x...

#### How to check the version of mefs?

> Run `mefs-user/keeper/provider version to see the version number of mefs

#### How can I check my role?

> During the startup process of mefs daemon, you can view the output prompt information;
> You can also run `mefs-user/keeper/provider test localinfo` to view your own role during operation.

## User

#### What is the LFS function of user?

> Mefs provides users with an encrypted file system LFS, which supports bucket and object operations

#### user 的 LFS 如何启动？

> After confirming that `mefs-user daemon` is running, and the running role is user, run `mefs-user lfs start`;
>
> When the return value is returned, there will be information about the success or failure of the startup. The initial startup process includes the entire network query and contract signing, so the startup time is relatively long.
>
> When the user starts lfs, he can choose to use the default parameters; if you lower the price parameter, you may not find enough providers;
>
> If you increase the ks parameter, you may not find enough keepers; if you increase the ps parameter, you may not find enough providers.

#### What are the LFS initialization errors of user?

* Input parameter error

> ps parameter setting should be greater than 1, other parameters should be greater than 0; rdo is true/false.

* Insufficient amount in user account

> The greater the storage duration and the storage size, the greater the amount required to sign the contract.

* What should I do if I cannot find enough number of keeper/provider after wrong parameter input?

> Restart LFS, set new parameters, and set rdo to true.

#### What are the common mistakes of LFS operation?

* When the bucket is created, the bucket already exists.

> This means that the bucket has been created with this name, and now the bucket is deleted, only marking records, not real deletion.
>
> Therefore, even if the bucket is deleted, creating it with this name again will still show that the bucket already exists.

* When downloading, it shows that the file already exists

> This is because the file already exists in the current directory, you can download it from another directory, or rename the file in the current directory.

* When uploading, it shows that the file already exists

> This means that there is already a file with this name in the currently uploaded bucket. Now that the file is deleted, it is only marked for recording, not a real deletion. So even if the file is deleted, creating it with this name again will still show that the file already exists. You can modify the upload path, for example, `mefs-user lfs put_object objectName bucketName/<new dir name>`,
> upload to the directory of < new dir name > corresponding to bucketName.

## Provider

#### What is the pos of the provider?

> The pos function is used for cold start. When the provider just joins the network, the amount of stored data is small.
>
> Pos generates local data according to the size of the mortgage space, and responds to the keeper challenge;
>
> When the provider receives the actual user data, it will gradually delete the pos data;
>
> The difference between pos data and actual user data is that the price of pos data is 1/10 of the default price, and the pos data will not be repaired.

#### How does the provider modify its price?

> When the provider is started, it resets its price parameters and sets rdo to true to update its storage price.

#### How does the provider set up its own master keeper?

> When the provider is running, it can set its own master keeper through `mefs contract addMasterKeeper 0x...`;
> The main keeper will give priority to providing its own provider and trigger the time and space payment in the upkeeping contract.

## others

#### Which keeper is in the system?

```
The following is the public keeper's account address:
0x1adCa07Ae9bC70fc8c8d4C972176d1a1C810f0Ec
0xE434216FDF5573D8334Cb65cA2Df053e8A6f76C5
0x0614bc4f711dC47Fb0BE3B3300CDcB3339F2dD88
0xE561B5EAB2B97FAba9965eCC0179848D317ec2D3
0xf904237239a79f535bdc77622CCfB31E3B3f83C9
0xd98FA04955365De35321478136fb8706049f7Ef9
0x071E7e6163B5855Fad5837BDDf1C50b70327074e
```

#### How to set the api address of the blockchain?

> Run `mefs-user/keeper/provider config Eth` to view the address of the blockchain to which you are connected,
> If you want to modify, you can run `mefs-user/keeper/provider config Eth xxx`, where xxx is the api address of the chain.
> The format is `http://ip:port`

#### How can I check my network connection status?

> Run `mefs-user/keeper/provider swarm peers` to view the nodes you are connected to.