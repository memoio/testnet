# MEFS Command line operation document

[TOC]

## Command details

### Start up

#### initialization

Mefs initialization, the default initialized directory is \$HOME/.mefs, you can set the initialized directory by export MEFS_PATH=<local dir>, and then run init.

```shell
> mefs-keeper init --netKey=<net key> --sk=<your private key> --pwd=<your password> --keyfile=<absolute path of your keyfile>
```

Parameter explanation:

- sk：private key address, if not specified, a new account will be created by default;
- pwd：password, the default password if not specified;
- keyfile: the full path of the keyfile file, if it is not specified, the default is to create a new keyfile;
- netKey: private network logo, now has dev and testnet; networks with the same logo can be interconnected, the default is dev.

#### Modify the network transmission port

The default is port 4001, if you need to change to <port num>, execute the following command;
Keeper requires that this <port num> can be directly accessed, or there is a port mapping on the external network machine.

```shell
// Execute before running daemon
mefs-keeper config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/<port num>\"]"
```

For example, change to port 4090:

```shell
// Execute before running daemon
mefs-keeper config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/4090\"]"
```

#### Start instance

```shell
> mefs-keeper daemon --netKey= <net key> --pwd=<your password> --sk=<your secretKey>
```

Parameter explanation:

- netKey: private network logo, now there are dev and testnet; networks with the same logo can be interconnected, the default is dev;
- pwd：password, the default password if not specified;
- sk：private key address, if not specified, it will be exported from the local private key file by default;

### View node information

Through `mefs-keeper id`, you can view the node's network address (8M...), account address (0x...), public key, communication address, and version information.

```shell
mefs-keeper id <peerID> --pwd=<your password>
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

Use `mefs-keeper info` to display information related to the keeper node.

#### Show revenue

mefs-keeper gains revenue by providing management services.

```shell
mefs-keeper info list_income
```
Output result:

```shell
manageIncome: 88155981.17 Gwei
posIncome: 0 Wei
```
manageIncome represents the income obtained through the management service, and posIncome represents the income obtained through the cold start management service.

#### Show other node addresses

In a storage service, the keeper signs a storage service contract with the user and the provider to jointly serve the user. The following commands can be used to query which users the keeper has signed a storage service contract with.

```shell
mefs-keeper info list_users
```

Output result:

```shell
8MJacEyv1Pf92d9k93uquFzPcnG5rH.fsID:8MG8vi4NGKmdrz3EuxAgrPatAk5LJ1 has keepers:8MH4Woxb2FkM5nFr86dHj21fLgEybi/8MJ5cAWfAP86cHmAcC3dxqzK41dh4a/8MK6qHvAfayLQy4d2684NTLAVLpQFk
8MJacEyv1Pf92d9k93uquFzPcnG5rH.fsID:8MG8vi4NGKmdrz3EuxAgrPatAk5LJ1 has providers:8MJJ58XHucLXfV5ej7GUXWhRDAHfim/8MGvZx9fireJe3McVXatDrim2CmNHJ/8MHdtmAtx5Q54So9MRWSSELorrZJdx/8MGXm9jEhY2JHAdpfbmbS4agqsJ2yM/8MKLYiSAFsdVxfj1PYNCQQjoG5Nwgm/8MH9yV7GZXhjYZeydVvZgEF3U7kxhN
```

This result shows that the keeper node and the user node (the network address is 8MJacEyv1Pf92d9k93uquFzPcnG5rH) have signed a storage service contract. 8MG8vi4NGKmdrz3EuxAgrPatAk5LJ1 represents the file system ID, which represents the storage service and is converted from the query contract address.

View the addresses of other keeper accounts that have signed a storage contract with the keeper node:

```shell
mefs-keeper info list_keepers
```

View the address of other provider accounts that have signed a storage contract with the keeper node:

```shell
mefs-keeper info list_providers
```

### View network nodes

#### View Keeper nodes in the network

Show all the Keeper account addresses registered on the chain:

```shell
mefs-keeper list keepers
```

Output result:

```shell
"KeeperCount": 3,
	"PledgeMoney": "30000000.00 Gwei", //keeper pledge total amount
	"OnlineCount": 2,//number of online keepers
	"OnlineKepepers": [ //keeper online
		{
			"Address": "0x489799F06C6E70599e1Cc34d393394D59Cca1695",//account address
			"Online": true,
			"PledgeMoney": "10000000.00 Gwei",//pledge amount
			"PledgeTime": "2020-09-28 Mon 13:01:29 CST"//pledge time
		},
		{
			"Address": "0xdB04F60f4313fdCb75C51679ad25dE5D5e92a45B",
			"Online": true,
			"PledgeMoney": "10000000.00 Gwei",
			"PledgeTime": "2020-09-28 Mon 13:05:10 CST"
		}
	],
	"OfflineCount": 1, //keeper not online
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

Show all Provider's account addresses registered on the chain:

```shell
mefs-keeper list providers
```
