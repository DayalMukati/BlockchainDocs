---
description: >-
  Here we are going to download fabric samples, binaries and docker images using
  shell scripts.
---

# 🧪 Fabric Installation & Test

### Download Fabric Network

Determine a location (mostly on the Desktop) on your machine where you want to place the _fabric-samples_ repository and enter that directory in a terminal window. The command that follows will perform the following steps:

1. If needed, clone the [hyperledger/fabric-samples](https://github.com/hyperledger/fabric-samples) repository
2. Check out the appropriate version tag
3. Install the Hyperledger Fabric platform-specific binaries and config files for the version specified into the /bin and /config directories of fabric-samples
4. Download the Hyperledger Fabric docker images for the version specified

Once you are ready, and in the directory into which you will install the Fabric Samples and binaries, go ahead and execute the command to pull down the binaries and images.



Let's Create a new folder on the Desktop:

```bash
mkdir hyperledger
cd hyperledger
```

Download samples, binaries and docker images:

```
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.3.0 1.5.2
```

The command above downloads and executes a bash script that will download and extract all of the platform-specific binaries you will need to set up your network and place them into the cloned repo you created above. It retrieves the following platform-specific binaries:

> * `configtxgen`,
> * `configtxlator`,
> * `cryptogen`,
> * `discover`,
> * `idemixgen`
> * `orderer`,
> * `peer`,
> * `fabric-ca-client`,
> * `fabric-ca-server`

and places them in the `bin` sub-directory of the current working directory.



### Test Fabric Network

We need to test our sample test network to make sure everything is working correctly.

1. Let's first go inside the test-network directory.

```bash
cd fabric-samples/test-network
```

Here we are going to use **`.network.sh`** shell script, which we help us in starting the network.&#x20;

&#x20;2\. Now Let's up the test-network:

```powershell
./network.sh up
```

You may check out running containers with below command.

```shell
docker ps
```

3\. We will create a channel after, starting a network in step 2.

```bash
./network.sh createChannel
```

4\. Deploy a sample chaincode for testing.

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

5\. Finally down the network.

```bash
./network.sh down
```
