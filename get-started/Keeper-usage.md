# Use The MEFS Testnet

## Operational Requirements

* Recommended configuration: 4 cores, 8G memory, 200GB storage, 20Mbps bandwidth;
* External network ip, port 4001 is usable;
* Docker environment;

## Get the Docker Image

```shell
> docker pull memoio/mefs-keeper:latest
```

## Generate Account

```shell
> docker run -it -v <your local storage dir>:/root --entrypoint="/app/create" memoio/mefs-keeper:latest
```

Then input the password of mefs-keeper(at least 8 digits, the default is "memoriae") as prompted, and the keyfile of mefs-keeper is stored in the < your local storage >/.mefs/keystore directory. 

An example of generating an account:

```shell
> docker run -it -v ~/docker-test/keeper:/root --entrypoint="/app/create" memoio/mefs-keeper:latest
> please input your password(at least 8): 12345678
> Private Key: 5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894
  Address: 0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0
```

The generated address is "0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0", the private key is "5cac2aaf3aa4c086a381cc0e74fdc3d685a99db5d320a2e0265ea426cf3d7894", the keyfile is stored in the ~/docker-test/keeper/.mefs/keystore directory, the file name is "0x32Ae578B69c2e3B484DEB01F6B5E65b9a61bC2a0", the password is "12345678".

## Register Keeper

Send an email to sup@memolabs.io to apply for keeper registration.

Email content: account address (such as 0x... generated above)、role (keeper)

## Start Keeper

Requirements: The 4001 port of the machine can be accessed by the public network, or there is a port mapping on the export machine.

### Start Command:

```shell
//Start docker; port 4001 is used for network connection
sudo docker run -d --stop-timeout 30 \
    -p <External Port Num>:<Port Num> \
    -v <storage dir>:/root \
    -e TRANSPORT=<Port Num> \
    -e WALLET="0x..." \
    -e PASSWORD="<your password>" \
    --mount type=bind,source="<keystore dir>",destination=/app/keystore \
    --name <countainer name> memoio/mefs-keeper:latest
```

### Parameter Explanation:

* TRANSPORT: < Port Num > is the network port of the program, the default is 4001; < External Port Num > is the port mapped out by docker, and the default can be the same as < Port Num >;

* WALLET: Account address (0x...), must be specified;
* PASSWORD: The password of the keyfile. If the keeper is running in the docker background mode, it must be specified; if it is running in the foreground mode, it can be entered during the running process;
* storage dir: data directory;
* keystore dir: The location of the keyfile exported after registration, the name of the keyfile is WALLET;

It takes about 20 minutes for the first startup.

### Log File:

In the *< storage dir >/.mefs* directory, there are startup log daemon.stdout.xx and the running log in the *logs* directory;

Through the running log, you can view the running status of the keeper node; when an error occurs, you can view the startup log;

## Enter the Terminal

```shell
//enter the docker
> sudo docker exec -it <container name> bash
```

The parameter explanation of each command can be found in the [documentation](/docs/cmd/mefs-keeper-command.md).