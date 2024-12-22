# 🚚 Chaincode Deployment

We are going to deploy FabCar chaincode on Test Network. Till now we have created a channel and our peers have joined the channel.&#x20;

1. We need to write a script to configure chaincode environment variables. Create a new script file inside `fabric-samples/test-network/scripts` and name it _`setFabCarGolangContext.sh`_

```sh
export CC_RUNTIME_LANGUAGE=golang
export CC_SRC_PATH="../chaincode/fabcar/go/"
export VERSION=1

echo Vendoring Go dependencies ...
pushd ../chaincode/fabcar/go
export GO111MODULE=on go mod vendor
popd
echo Finished vendoring Go dependencies
```

2. Update the environment variable to configure the use of GoLang Chaincode.

```shell
source ./scripts/setFabCarGolangContext.sh
export FABRIC_CFG_PATH=$PWD/../config/
export FABRIC_CFG_PATH=${PWD}/configtx
export CHANNEL_NAME=mychannel
export PATH=${PWD}/../bin:${PWD}:$PATH
```

3. **Package the chaincode:** Chaincode needs to be packaged in a tar file before it can be installed on your peers. You can package a chaincode using the Fabric peer binaries, the Node Fabric SDK, or a third-party tool such as GNU tar.

```sh
source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode package asset.tar.gz --path ${CC_SRC_PATH} --lang ${CC_RUNTIME_LANGUAGE} --label asset_${VERSION}
```

Check if the package is created: ‘fabcar.tar.gz’ file should be seen.

4.  **Install the Chaincode:** You need to install the chaincode package on every peer that will execute and endorse transactions. You need to complete this step using your Peer Administrator, whether using the CLI or an SDK.&#x20;

    A successful install command will return a chaincode package identifier, which is the package label combined with a hash of the package. This package identifier associates a chaincode package installed on your peers with a chaincode definition approved by your organization.

    \


    1. Install the chaincode on the peer of Org1

```sh
peer lifecycle chaincode install asset.tar.gz
```

2. Install the chaincode on peer of Org2

```sh
source ./scripts/setOrgPeerContext.sh 2
peer lifecycle chaincode install asset.tar.gz
```

5. The Query for Installed package

```sh
peer lifecycle chaincode queryinstalled 2>&1 | tee outfile
```

6. Set the PACKAGE\_ID value

Create a new script file inside the _`scripts`_ folder named: _`setPackageID.sh`_ and write below code inside it.

```sh
#!/bin/bash

PACK_ID=$(sed -n "/asset_${VERSION}/{s/^Package ID: //; s/, Label:.*$//; p;}" $1)
export PACKAGE_ID=$PACK_ID
echo $PACKAGE_ID
```

Now run the script by below cmd.

```sh
source ./scripts/setPackageID.sh outfile
```



7. **Approval Process:** The chaincode is governed by a chaincode definition. When channel members approve a chaincode definition, the approval acts as a vote by an organization on the chaincode parameters it accepts. These approved organization definitions allow channel members to agree on a chaincode before it can be used on a channel.

\


Approve the Chaincode as Org1

```sh
source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name asset --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}
```

Check for committedness as Org1

```sh
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name asset --version ${VERSION} --sequence ${VERSION} --output json --init-required
```

Check for committedness as Org2

```sh
source ./scripts/setOrgPeerContext.sh 2
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name asset --version ${VERSION} --sequence ${VERSION} --output json --init-required

```

Approve the Chaincode as Org2

```sh
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name asset --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}
```

Check for committedness as Org2

```sh
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name asset --version ${VERSION} --sequence ${VERSION} --output json --init-required
```

Check for committedness as Org1

```sh
source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode checkcommitreadiness --channelID $CHANNEL_NAME --name asset --version ${VERSION} --sequence ${VERSION} --output json --init-required

```

8. **Set the peer address for identifying the endorsing peers**

Create a new script in the `scripts` the folder named: `setPeerConnectionParam.sh`

```sh
#!/bin/bash

source scripts/envVar.sh

parsePeerConnectionParameters $@

echo ${PEER_CONN_PARMS[@]}

export PEER_CONN_PARAMS=${PEER_CONN_PARMS[@]}
```

Now run the script.

```sh
source ./scripts/setPeerConnectionParam.sh 1 2
```

9. **Commit the chaincode definition to Channel**

```sh
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name asset $PEER_CONN_PARAMS --version ${VERSION} --sequence ${VERSION} --init-required
```



10. Check docker status

```sh
docker ps
```

11. Query chaincode commit as Org2

```sh
peer lifecycle chaincode querycommitted --channelID $CHANNEL_NAME --name asset

```

13. Query chaincode commit as Org1

```sh
source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode querycommitted --channelID $CHANNEL_NAME --name asset

```

\
