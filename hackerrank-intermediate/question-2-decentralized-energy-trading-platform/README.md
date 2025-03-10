# Question 2: Decentralized Energy Trading Platform

### **Problem Statement: Decentralized Energy Trading Platform**

You are tasked with implementing a **peer-to-peer energy trading system** using Hyperledger Fabric. The goal is to allow **users to register, manage energy units, and trade energy using tokens**.

Your smart contract should support the following functionalities:

**Register a user** (`RegisterUser`): A user can be a **producer** or a **consumer**, and they start with a specific balance of tokens and energy units.\
**Produce energy** (`ProduceEnergy`): A producer can generate energy, increasing their available energy units.\
**Transfer energy** (`TransferEnergy`): A consumer can purchase energy from a producer by transferring tokens.\
**Get user details** (`GetUserDetails`): Retrieve the full details of a user, including balance and energy units.

**Test the Smart Contract by Performing the Following Actions:**

1. **Register a producer (`producer1`)** with **1000 tokens** and **0 energy units**.
2. **Register a consumer (`consumer1`)** with **500 tokens** and **0 energy units**.
3. **Producer (`producer1`) generates 100 energy units**.
4. **Consumer (`consumer1`) purchases 50 energy units from producer (`producer1`) for 100 tokens**.
5. **Check updated balances** and energy units for both producer and consumer.

**Requirements:**

* The chaincode should be written in **Go (Golang)**.
* The **test network** should be used to test the contract, and test results should be displayed using `peer chaincode invoke` and `peer chaincode query`.

***

#### **Deployment & Testing Commands**

**Deploy the Chaincode**

```bash
cd fabric-samples/test-network
sudo ./network.sh deployCC -c mychannel -ccn energycc -ccp ../chaincode -ccl go
```

**Set Environment for Org1**

```bash
source ./scripts/setOrgPeerContext.sh 1
export FABRIC_CFG_PATH=${PWD}/configtx
```

**Execute Transactions**

**1. Register a Producer and Consumer**

Register a **producer (`producer1`)** with 1000 tokens and a **consumer (`consumer1`)** with 500 tokens.

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n energycc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterUser", "producer1", "Producer", "1000"]}'


peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n energycc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterUser", "consumer1", "Consumer", "500"]}'

```

**2. Check use Balance**

Verify that `producer1` and `consumer1` were created with correct balances.

```bash
peer chaincode query -C mychannel -n energycc -c '{"Args":["GetUserDetails","producer1"]}'
peer chaincode query -C mychannel -n energycc -c '{"Args":["GetUserDetails","consumer1"]}'

```

**3. Producer Generates Energy**

`producer1` generates **100 energy units**.

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n energycc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ProduceEnergy", "producer1", "100"]}'

```

**4. Consumer Purchases Energy from Producer**

`consumer1` buys **50 energy units** from `producer1` for **100 tokens**.

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n energycc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["TransferEnergy", "producer1", "consumer1", "50", "100"]}'
```

5.  **Verify Updated Balances and Energy Units**

    Check if the **balances** and **energy units** have been updated correctly.

    ```bash
    peer chaincode query -C mychannel -n energycc -c '{"Args":["GetUserDetails","producer1"]}'
    peer chaincode query -C mychannel -n energycc -c '{"Args":["GetUserDetails","consumer1"]}'
    ```

***
