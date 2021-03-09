# MEFS Command Line Operation Document

[TOC]

## Command details

### start up

#### initialization

Mefs initialization, the default initialized directory is \$HOME/.mefs, you can set the initialized directory by export MEFS_PATH=<local dir>, and then run init.

```shell
> mefs-user init --netKey=<net key> --sk=<your private key> --pwd=<your password> --keyfile=<absolute path of your keyfile>
```

Parameter explanation:

- sk：private key address;
- pwd：password;
- keyfile: the full path of the keyfile file;
- netKey: private network symbol, now has dev and testnet; networks with the same symbol can be interconnected.

#### Modify the network transmission port

The default is port 4001, if you need to change to <port num>, execute the following command:

```shell
// execute before running daemon
mefs-user config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/<port num>\"]"
```

For example, change to port 4090:

```shell
// execute before running daemon
mefs-user config --json Addresses.Swarm "[\"/ip4/0.0.0.0/tcp/4090\"]"
```

#### Startup example

```shell
// Run in the background
> mefs-user daemon --netKey=<net key> --pwd=<your password> >> log 2>&1 &
```

Parameter explanation:

- pwd：password;
- netKey: private network symbol, now has dev and testnet.

#### Start user's LFS

After starting the mefs daemon, the user starts its storage space.

Due to the need to match the contract and deploy the contract, it takes about 30 minutes to start.

```shell
> mefs-user lfs start <public address> --sk=<private key> --pwd=<password> --dur=<duration> --cap=<capacity> --price=<price> --ks=<keeper SLA> --ps=<provider SLA>
```

Parameter explanation:

- public address：account address（0x...）；local account's public address by default;
- sk：private key address； import from local private key file by default;
- pwd：password; DEFAULTPASSWORD by default;
- dur：the storage time provided; calculated in days, the default is 100;
- cap：provided storage size; calculated in MB, the default is 1TB;
- price：the storage price provided, calculated by weiDollar/MB/hour, and the default is 4 * 10^9; about 3 USD/(TB\*month);
- ks：the number of keeper required, the default is 3;
- ps：the number of provider required, the default is 6;

### Use LFS（cli）

MEFS provides each user with a dedicated encrypted storage space (LFS). Each storage space contains multiple buckets. A bucket is a container for users to store objects. Each bucket contains multiple objects ( object), we can think of objects as files. The redundancy policy of the bucket can be specified when it is created (all objects stored in the bucket use this redundancy policy), and the object data is encrypted using symmetric encryption.

The user can operate data through command line, network (http), and gateway.

It can also be operated through the sdk, and only the go version is currently available.

#### bucket operation

- create bucket

command description：`create_bucket` creates a bucket based on the name of BucketName. Each bucket can be set with different redundancy strategies. The redundancy strategy is multiple replicas or erasure codes (Reed-Solomon Codes). The number of data blocks and check blocks can be adjusted to determine the level of redundancy. The erasure codes of 3 data blocks and 2 check blocks are used by default, and the loss of two blocks can be tolerated.

```shell
> mefs-user lfs create_bucket <BucketName> --policy=<redundancy> --dc=<data count> --pc=<parity count> --addr=<public key>
```

command description:

```shell
--BucketName: name of bucket, minimum 3 bytes and maximum 256 bytes;
--policy：redundancy strategy, --policy=1 means to use erasure codes, --policy=2 means to use multiple copies, and erasure codes are used by default;
--dc：the number of data blocks, the default is 3;
--pc：the number of check blocks, the default is 2; when using the multiple copy strategy, the actual data block is 1, and the check block is dc+pc-1;
--addr: the address of user, which is empty by default, indicating that is the address of the local node.
```

result output:

```shell
Method: Create Bucket    // command name
BucketName: <BucketName> // name of created bucket
--BucketID: <BucketID>  // internal ID of bucket
--Ctime: <Ctime> // create time
--Policy: <Policy> // redundancy strategy
--DataCount: <DataCount> // number of data block
--ParityCount: <ParityCount>  // number of check blocks
```

- Bucket List

command description：`list_buckets` display all buckets created by this user, including the name (BucketName), creation time (Ctime), redundancy policy (Policy) and redundancy parameters (DataCount, ParityCount) of each bucket.

```shell
> mefs-user lfs list_buckets --addr=<public key>
```

Parameter explanation:

```shell
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

result output:

```shell
Method: List Buckets
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
--Policy: <Policy>
--DataCount: <DataCount>
--ParityCount: <ParityCount>
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
...
```

- Bucket Information

command description: If the BucketName exists, `head_bucket` displays the creation time, redundancy strategy and redundancy parameters of this bucket; if it does not exist, returns the bucket does not exist.

```shell
> mefs-user lfs head_bucket <BucketName> --addr=<public key>
```

Parameter explanation:

```shell
BucketName：name of bucket
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

result output:

```shell
Method: Head Bucket
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
--Policy: <Policy>
--DataCount: <DataCount>
--ParityCount: <ParityCount>
```

- Delete Bucket

command description: if the bucket named exists, `delete_bucket` deletes the bucket; if it does not exist, returns the bucket does not exist. Only delete when the bucket is empty, otherwise it returns that bucket is not empty.

```shell
> mefs-user lfs delete_bucket <BucketName> --addr=<public key>
```

Parameter explanation:

```shell
BucketName：name of bucket
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

result output:

```shell
Method: Delete Bucket
BucketName: <BucketName>
--BucketID: <BucketID>
--Ctime: <Ctime>
--Policy: <Policy>
--DataCount: <DataCount>
--ParityCount: <ParityCount>
```

#### object operation

- upload object

command description: `put_object` uploads an object named ObjectName to the bucket named BucketName; if the bucket does not exist, it returns bucket does not exist; if the object already exists, it returns object already exists.

```shell
> mefs-user lfs put_object <ObjectName> <BucketName> --addr=<public key>
```

Parameter explanation:

```shell
ObjectName：the name of the uploaded file, using a relative or absolute path;
BucketName：name of bucket;
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

result output:

```shell
Method: Put Object
ObjectName: <ObjectName>  // name of object
--ObjectSize: <ObjectSize> // the size of the uploaded object
--MD5: <MD5>               // the hash value of the uploaded object by MD5 encryption
--Ctime: <Ctime>           // create time of object
--Dir: <Dir>               // whether it is a directory, true indicates yes, false indicates no.
--LatestChalTime:<LatestChalTime>  // the time this object was last challenged
```

- download object

command description: `get_object` downloads an object named ObjectName from the bucket named BucketName; if the bucket does not exist, returns the bucket does not exist; if the object does not exist, returns the object does not exist.

```shell
mefs-user lfs get_object <BucketName> <ObjectName> --o=<OutputName> --addr=<public key>
```

Parameter explanation:

```shell
ObjectName：name of object that want to get;
BucketName：name of bucket;
--o: the default name of  download object is ObjectName, if this parameter is set, it is OutputName.
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

Result output:

Downloaded file

- File List

command description: `list_objects` lists all the objects in the bucket named BucketName, including the object size, creation time, MD5 value, whether it is a directory, and the time of the most recent challenge.

```shell
mefs-user lfs list_objects <BucketName> --addr=<public key>
```

Parameter explanation:

```shell
BucketName：name of bucket that want to list；
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

Result output:

```shell
Method: List Object
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--MD5: <MD5>
--Ctime: <Ctime>
--Dir: <Dir>
--LatestChalTime:<LatestChalTime>
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--MD5: <MD5>
--Ctime: <Ctime>
......
```

- File Imformation

command description: `head_object` displays the size of the object named ObjectName in the bucket named BucketName, the MD5 value, the creation time, whether it is a directory, and the time of the most recent challenge.

```shell
mefs-user lfs head_object <BucketName> <ObjectName> --addr=<public key>
```

Parameter explanation:

```shell
ObjectName：name of object that want to head;
BucketName：name of bucket;
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

Result output:

```shell
Method: Head Object
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--MD5: <MD5>
--Ctime: <Ctime>
--Dir: <Dir>
--LatestChalTime:<LatestChalTime>
```

- Delete File

command description: `delete_object` deletes the object named ObjectName from the bucket named BucketName.

```shell
 mefs-user lfs delete_object <BucketName> <ObjectName> --addr=<public key>
```

Parameter explanation:

```shell
ObjectName：name of object to delete;
BucketName：name of bucket that object is in；
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

Result output:

```shell
Method: Delete Object
ObjectName: <ObjectName>
--ObjectSize: <ObjectSize>
--Ctime: <Ctime>
--Dir: <Dir>
--LatestChalTime:<LatestChalTime>
```

#### Operation about Role

- List the corresponding keeper

Command description: `list_keepers` lists the keeper who signed the UpKeeping contract with this user

```shell
mefs-user lfs list_keepers
```

The output is the corresponding keeper id.

- List the corresponding provider

Command description: `list_providers` lists providers that store data for this user.

```shell
mefs-user lfs list_providers
```

The output is the corresponding provider id.

#### Others

- Refresh metadata

Command description: `fsync` manually refreshes the metadata of lfs. This command is executed on the keeper. Metadata includes SuperBlock, BucketInfo, ObjectInfo.

```shell
mefs-user lfs fsync
```

The output is 'Flush Success'.

- Check the storage space used

Command description: `show_storage` queries the user's used space, that is, how much data is stored in all buckets of the user, the unit is kb.

```shell
mefs-user lfs show_storage --addr=<public key>
```

Parameter explanation:

```shell
--addr: user address, which is empty by default, indicating that is the address of the local node.
```

The output is the corresponding space, the format is two decimal places with unit (kb).

### http Operation

Mefs commands can all be operated using http.

#### Configuration

Before mefs-user starts, perform the following configuration:

```shell
// mefs api port setting, default is 5001
mefs-user config Addresses.API /ip4/0.0.0.0/tcp/5001
// cross-domain access
mefs-user config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]'
mefs-user config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST"]'
```

Then restart mefs-user to use http to operate.

#### Usage

A command similar to the following:

```shell
mefs-user rootcmd subcmd <arg1> <arg2> -opname1=<op1> -opname2=<op2>
```

The corresponding http request is:

```shell
curl  "http://<ip>:<port>/api/v0/api/v0/<rootcmd>/<subcmd>?arg=<arg1>&arg=<arg2>&opname1=<op1>&opname2=<op2>"
```

ip is the network address of the machine where mefs-user is started. The port defaults to 5001. If cross-domain access is configured before running, you can use the external network ip to access, otherwise you can only access it through 127.0.0.1.

#### example

- Show all bucket information

```shell
curl  "http://127.0.0.1:5001/api/v0/lfs/list_buckets?addr=<public key>"
```

The output is in standard json format:

```json
{
  "Method": "List Buckets",
  "Buckets": [
    {
      "BucketName": "<BucketName>",
      "BucketID": "<BucketID>",
      "Ctime": "<Ctime>",
      "Policy": "<Policy>",
      "DataCount": "<DataCount>",
      "ParityCount": "<ParityCount>"
    },
    {
      "BucketName": "<BucketName>",
      "BucketID": "<BucketID>",
      "Ctime": "<Ctime>",
      "Policy": "<Policy>",
      "DataCount": "<DataCount>",
      "ParityCount": "<ParityCount>"
    }
  ]
}
```

- Display information about all objects in a bucket

```shell
curl  "http://127.0.0.1:5001/api/v0/lfs/list_objects?arg=<BucketName>&addr=<public key>"
```

The output is in standard json format:

```json
{
  "Method": "List Objects",
  "Objects": [
    {
      "ObjectName": "<ObjectName>",
      "ObjectSize": "<ObjectSize>",
      "Ctime": "<Ctime>",
      "Dir": "<Dir>",
      "LatestChalTime": "<LatestChalTime>"
    },
    {
      "ObjectName": "<ObjectName>",
      "ObjectSize": "<ObjectSize>",
      "Ctime": "<Ctime>",
      "Dir": "<Dir>",
      "LatestChalTime": "<LatestChalTime>"
    }
  ]
}
```

### Gateway mode

Multiple users can share one mefs's running program.

#### user agent start

After mefs is started, other users can also be started as a proxy.

```shell
mefs-user lfs start <addr> --sk=<private key> --pwd=<password>
```

Parameter explanation:

```
addr：account address;
--sk：the private key of the user; if the private key corresponding to the address does not match, the address of the private key shall prevail;
--pwd：user's password;
```

#### user agent stop

After mefs is started, other users can also be shut down by proxy.

```go
mefs-user lfs kill addr --pwd=<password>
```

Parameter explanation:

```shell
addr：account address
--pwd：account password
```

#### Usage

- cli

```shell
mefs-user rootcmd subcmd arg1 op1=arg2 --addr=<public key>
```

- http

```shell
curl  "http://<ip>:5001/api/v0/api/v0/<roocmd>/<subcmd>?arg=<arg1>&op1=<arg2>&addr=<public key>"
```

#### example

The user whose address is the public key obtains the file named ObjectName from the BucketName bucket.

- cli

```shell
mefs lfs get_object <BucketName> <ObjectName> --addr=<public key>
```

- http
```shell
curl  "http://127.0.0.1:5001/api/v0/lfs/get_object?arg=<BucketName>&arg=<ObjectName>&addr=<public key>"
```
