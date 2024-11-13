---
description: We will try to understand the Test Network in-depth.
---

# ✈️ Launch Network

Test Network will be having below steps to be performed in sequence.

1. Certificate Generation - CryptoGen Tool/Fabric CA
2. Start the docker network - HLF Test Network
3. Create a channel - Orderer will create it. (My channel)
4. Peers will join the channel.
5. Anchor peer update
6. Chaincode Deployment: It has a lifecycle (Chaincode Lifecycle)
   1. Packaging&#x20;
   2. Install
   3. Approve&#x20;
   4. Commit
7. Init the ledger&#x20;
8. Invoke or Query Transactions&#x20;

### Note: Please Run these cmds[^1] before step 1.

```
cd Desktop/hyperledger/fabric-samples/test-network/
./network.sh down

docker ps -aq 
docker volume ls
```

### If there are containers then run :

```
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
docker volume prune
```

## Test Network Execution:&#x20;

### Certificate Generation Using CryptoGen Tool:

In the test network folder do the following:

1. Set the Config Path

```bash
export PATH=${PWD}/../bin:${PWD}:$PATH
```

2. Create the Certificates for Org1

```bash
cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output="organizations"
```

3. Create the certificates for Org2

```bash
cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output="organizations"
```

4. Create Certificates for OrdererOrg

```bash
cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output="organizations"
```

### Start Docker Network

5. Start the docker containers

```bash
export COMPOSE_PROJECT_NAME=net
IMAGE_TAG=latest docker-compose -f docker/docker-compose-test-net.yaml up

```

6. Open a new Terminal
7. Check the docker containers running status

```bash
docker ps
```

[^1]: 
