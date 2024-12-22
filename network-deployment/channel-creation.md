---
description: >-
  Here we will understand the Confixtx.yaml file and learn how to create
  channel.
---

# ⛓️ Channel Creation

### Create Channel

1. Set the Config Path

```bash
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
export CHANNEL_NAME=mychannel
```

2. Create the System Genesis  Block and Channel Genesis block

```
configtxgen -profile ChannelUsingRaft -outputBlock ./channel-artifacts/${CHANNEL_NAME}.block -channelID $CHANNEL_NAME
```

3. Convert Block to JSON format to understand the data inside it.

```sh
configtxgen -inspectBlock ./channel-artifacts/mychannel.block > dump.json
```

4. Copy some prerequisites

```shell
cp ../config/core.yaml ./configtx/.
```

5. Create the Channel

Let's Export the Environment variables to provide Orderer access to our terminal.

```
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
```

Let's create a channel using "OSNADMIN" binary file.

```
osnadmin channel join --channel-id $CHANNEL_NAME --config-block ./channel-artifacts/${CHANNEL_NAME}.block -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"
```

6. Join the Organizational peers in the channel&#x20;

#### First Let's join Org1 Peer.

To join the channel, our terminal needs access to peer, for that let's write a script which will set the environment variables.

Script Name: **setOrgPeerContext.sh**

```sh
#!/bin/bash

# import utils
source scripts/envVar.sh

ORG=$1

setGlobals $ORG
```

Now let's source the script.

```sh
source ./scripts/setOrgPeerContext.sh 1
```

Let's join the peer now

```sh
peer channel join -b ./channel-artifacts/mychannel.block
```

#### Join Org2 Peer Now

```sh
source ./scripts/setOrgPeerContext.sh 2
peer channel join -b ./channel-artifacts/mychannel.block
```

7. Update Anchor peer for Org1

```sh
source ./scripts/setOrgPeerContext.sh 1
docker exec cli ./scripts/setAnchorPeer.sh 1 $CHANNEL_NAME
```

8. Update Anchor peer for Org2

```sh
source ./scripts/setOrgPeerContext.sh 2
docker exec cli ./scripts/setAnchorPeer.sh 2 $CHANNEL_NAME
```

That's it!\
