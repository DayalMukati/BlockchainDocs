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

To install Hyperledger Fabric, we need to install the script. Let’s download it using the cURL command.\


```
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
```

Let’s verify if the script has been downloaded successfully.&#x20;

```
ls
```

install-fabric.sh

Run the script with the -h option to see the options available:

```
 ./install-fabric.sh -h
```



You should see the following output:

`Usage: ./install-fabric.sh [-f|--fabric-version <arg>] [-c|--ca-version <arg>] <comp-1> [<comp-2>] ... [<comp-n>] ...`

&#x20;       `<comp>: Component to install one or more of  d[ocker]|b[inary]|s[amples]. If none specified, all will be installed`

&#x20;       `-f, --fabric-version: FabricVersion (default: '2.5.6')`

&#x20;       `-c, --ca-version: Fabric CA Version (default: '1.5.9')`

#### Download Samples Using the Script

To specify the components to download, add one or more of the following arguments. Each argument can be shortened to its first letter.

\


* docker to use Docker to download the Hyperledger Fabric container images
* podman to use Podman to download the Hyperledger Fabric container images
* binary to download the Hyperledger Fabric binaries
* samples to clone the Hyperledger fabric-samples GitHub repository to the current directory

\


To pull the Docker containers and clone the samples repository, run one of the following commands:

```
./install-fabric.sh docker samples binary
```

If no arguments are supplied, then the arguments docker binary samples are assumed.



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
