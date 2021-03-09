# MEFS Command line operation document

[TOC]

## Command details

### Start up

#### initialization

Mefs initialization, the default initialized directory is \$HOME/.mefs, you can set the initialized directory by export MEFS_PATH=<local dir>, and then run init.

```shell
> mefs-provider init --netKey=<net key> --sk=<your private key> --pwd=<your password> --keyfile=<absolute path of your keyfile>
```

Parameter explanation:

- sk：private key address;
- pwd：password;
- keyfile: the full path of the keyfile file;
- netKey: private network logo, now has dev and testnet; networks with the same logo can be interconnected.

#### Modify the network transmission port

The default is port 4001, if you need to change to <port num>, execute the following command;
The provider requires that this <port num> can be directly accessed, or there is a port mapping on the external network machine.

```shell
// execute before running daemon
mefs-provider config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/<port num>\"]"
```

For example, change to port 4090:

```shell
// execute before running daemon
mefs-provider config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/4090\"]"
```

#### Start instance

```shell
> mefs-provider daemon --netKey= <net key> --pwd=<your password> --dur=<your storage duration> --cap=<your storage capacity> --price=<your storage price> --deCap=<your deposit capacity> --rdo=<bool> --pos=true
```

Parameter explanation:

- dur: the length of storage time provided; calculated in days, the default is 365 days;
- cap：the size of the storage space provided; calculated in MB, the default is 1TB;
- price: the storage price provided is calculated in wei, and the default is 4 * 10^9 wei; that is, 3 USD/(TB\*month);
- rdo：whether to redeploy the offer contract, the default is false;
- deCap：staking space; calculated in MB, 1TB by default;
- pos：whether to use the cold start function, the default is false and not turned on; if you use it, you need to ensure that the available space is greater than the pledged space;
- netKey: private network logo, now there are dev and testnet; networks with the same logo can be interconnected;

### View node information

Through `mefs-provider id`, you can view the node's network address (8M...), account address (0x...), public key, communication address, and version information.

```shell
mefs-provider id <peerID> --pwd=<your password>
```
Parameter explanation:

* peerID：the ID of the node to be searched; the default is the local node;
* pwd：the password of the node is DefaultPassword by default, and the proxy node is empty by default;

Result output:

```shell
{
	"NetworkAddr": "8MJ5cAWfAP86cHmAcC3dxqzK41dh4a",
	"AccountAddr": "0x91b90d08aaE86F87648cF6B497918D487A5C19Ef",
	"PublicKey": "A6nefy7/FZbrrszX/itXdKmNTrFmkwMSYptFYwBjRVxz",
	"Addresses": [
		"/ip4/127.0.0.1/tcp/4001/p2p/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a",
		"/ip4/172.26.133.70/tcp/4001/p2p/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a",
		"/ip4/172.17.0.1/tcp/4001/p2p/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a"
	],
	"AgentVersion": "go-mefs/v0.3.2/ddf22678"
}
```

### Display information

#### Local node information

```shell
mefs-provider info self
```

Output result:

```shell
"Wallet": "0xa12626B388dc08481520B0f9c7009b9505f87f5e", //account address
	"StartTime": "2020-11-16 Mon 10:24:50 CST", //registration time
	"UpTime": "45 day 7 hour 42 minute 10 second", //online time
	"ReadyForService": true, //whether the service has been started
	"PublicNetwork": "/ip4/58.210.46.6/tcp/4115/p2p/8MJJ58XHucLXfV5ej7GUXWhRDAHfim", //public address
	"PublicReachable": true,
	"Balance": "10874.62 Token",
	"PledgeBytes": "1.00 TiB",
	"UsedBytes": "639.31 GiB",
	"PosBytes": "0 B",
	"LocalFreeBytes": "232.77 GiB",
	"OfferAddress": "0x8bc53304465Aa7214Baaf4407f6d4626E15bBab0",
	"OfferCapacity": "976.56 GiB",
	"OfferPrice": "3.02 Dollar/(TiB*Month) (For now, 1 Dollar = 100 Token)",
	"OfferDuration": "8640000 day",
	"OfferStartTime": "2020-05-15 Fri 04:38:38 CST",
	"TotalIncome": "2.14 Token",
	"StorageIncome": "1.51 Token",
	"DownloadIncome": "7498443.30 Gwei",
	"PosIncome": "624882551.42 Gwei"
```

#### user information

```shell
mefs-provider info users
```

Output result:

```shell
8MJg6VvVqYPFr2d38wxUVGRZqqJcnv/8MGXdiJq3AhHuQa3dh1pucP6rJXahb
8MKWbYg3b9Gxj2c7szWkzSxAJS3V6y/8MHLoGHF2rBXPocBztUu2tAEaiqYqb
8MJr4bwoDcsGVdD7hhAJYMs8FkN3Kt/8MKACLbaSreLC2e7nkfEjPZvbeLhzk
8MKDyauiihGANvuAxc5G3N6B922rFy/8MKbFLenGA3zbNmKiZvkWaoztGLvJu
```
The first ID represents the User with whom the Provider node signs a storage service contract, and the first represents the file system ID.

#### group information

Provider provides storage services for users to form a group. The group information contains detailed information about storage services.

```shell
mefs-provider info group <uid> <qid>
```

Parameter explanation:

* uid：ID of the user;
* qid：ID of the file system;

### View network nodes

#### View Keeper nodes in the network

Show all the Keeper account addresses registered on the chain:

```shell
mefs-provider list keepers
```

Output result:

```shell
"KeeperCount": 3,
	"PledgeMoney": "30000000.00 Gwei", //keeper pledge total amount
	"OnlineCount": 2, //number of online keepers
	"OnlineKepepers": [ //Keeper online
		{
			"Address": "0x489799F06C6E70599e1Cc34d393394D59Cca1695",//account address
			"Online": true,
			"PledgeMoney": "10000000.00 Gwei", //pledge amount
			"PledgeTime": "2020-09-28 Mon 13:01:29 CST" //Pledge time
		},
		{
			"Address": "0xdB04F60f4313fdCb75C51679ad25dE5D5e92a45B",
			"Online": true,
			"PledgeMoney": "10000000.00 Gwei",
			"PledgeTime": "2020-09-28 Mon 13:05:10 CST"
		}
	],
	"OfflineCount": 1, //Keeper not online
	"OfflineKeepers": [
		{
			"Address": "0x91b90d08aaE86F87648cF6B497918D487A5C19Ef",
			"Online": false,
			"PledgeMoney": "10000000.00 Gwei",
			"PledgeTime": "2020-09-28 Mon 12:54:27 CST"
		}
	]
```
#### View Provider nodes in the network

Show all Provider account addresses registered on the chain:

```shell
mefs-provider list providers
```