# Question 1: Write and Deploy Land Registration Chaincode

You are tasked with developing a **simple land registration smart contract** for a **Hyperledger Fabric network**. The goal is to create a smart contract that **registers land, transfers ownership, and retrieves land details**.

You are required to:

Write a smart contract that supports the following functionalities:

1. **RegisterLand(landID, ownerName, location)** → Register a new land record with a **unique land ID**, owner, and location.
2. **TransferLandOwnership(landID, newOwner)** → Transfer the ownership of a land record to a new owner.
3. **GetLandDetails(landID)** → Retrieve the details of a specific land record.

#### **Deployment Environment**

Use the provided **pre-configured test network**, which includes:

* **Two peers**, one for `Org1` and one for `Org2`.
* **A default channel** named `mychannel`.
* **A pre-initialized ledger with no data**.

#### **Test the Smart Contract** by performing the following actions:

**Register a land record** with:

* `landID`: `land1`
* `ownerName`: `Charlie`
* `location`: `NYC`

**Transfer ownership** of `land1` from `Charlie` to `David`.

**Retrieve land details** to confirm that the **new owner** is `David`.

***

#### **Deployment Steps**

**Deploy the Chaincode**

```bash
cd fabric-samples/test-network
sudo ./network.sh deployCC -c mychannel -ccn landcc -ccp ../chaincode -ccl go
```

**Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

**Execute Transactions**

**1. Register a Land Record**

```bash
peer chaincode invoke -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls --cafile $ORDERER_CA -C mychannel -n landcc \
  --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
  --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
  -c '{"Args":["RegisterLand","land1","Charlie","NYC"]}'
```

**2. Transfer Land Ownership**

```bash
peer chaincode invoke -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls --cafile $ORDERER_CA -C mychannel -n landcc \
  --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
  --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
  -c '{"Args":["TransferLandOwnership","land1","David"]}'
```

**3. Retrieve Land Details**

```bash
peer chaincode query -C mychannel -n landcc -c '{"Args":["GetLandDetails","land1"]}'
```

***

#### **Notes:**

The following steps are already performed for you as prerequisites:

The **Hyperledger Fabric test network** can be started using the command:

```bash
./network.sh up createChannel
```

with the default channel **mychannel**.

**Peers for Org1 and Org2** are already joined to the channel.

The **chaincode environment** setup is pre-configured in the **chaincode directory**.
