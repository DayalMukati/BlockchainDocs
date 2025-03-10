# Question 3: Adding a New Peer to Org1 & Deploying Intellectual Property Chaincode

**Decentralized Intellectual Property (IP) Rights Management System using Hyperledger Fabric**

Your organization is building a decentralized Intellectual Property (IP) rights management system using Hyperledger Fabric. The goal is to allow creators to register IP, transfer rights, and verify ownership in an immutable, trustless system.

***

### **Enhancing Scalability and Redundancy**

To enhance scalability and redundancy, you need to:

* Add a new peer (**peer1.org1.example.com**) to **Org1**.
* Create a new custom channel (**ipchannel**) to manage IP transactions separately.
* Deploy the **IP Rights Management Chaincode (ipcc)** on the new channel.

***

### **Tasks to be Completed**

* Add a new peer (**peer1.org1.example.com**) to **Org1**, running on port **8051**.
* Configure **Org1’s MSP** and update **docker-compose** settings for the new peer.
* Start the new peer and join it to **ipchannel**.
* Deploy the **Intellectual Property Chaincode (ipcc)** on **ipchannel**.
* Test transactions by registering an **IP**, transferring ownership, and verifying ownership.

***

### **Steps to be Performed**

#### **Add a New Peer (peer1.org1.example.com) to Org1**

* Modify **crypto-config.yaml** to define the new peer.
* Use **cryptogen** to generate crypto materials for the new peer.
* Update **docker-compose** files to include **peer1.org1.example.com**.
*   Start the new peer using:

    ```bash
    docker-compose up -d
    ```
* Join **peer1.org1.example.com** to **ipchannel**.

#### **Deploy the Chaincode**

* Deploy **ipcc** onto **ipchannel**.
* Ensure endorsement policy includes both **Org1 peers (peer0 & peer1)** and **Org2**.

#### **Smart Contract Implementation**

The chaincode should support:

* Registering an **Intellectual Property (IP)**.
* Transferring ownership of **IP** between parties.
* Verifying **IP** ownership.
* Retrieving **IP** details.

#### **Testing & Validation**

* Register an IP (**ip1**) for a creator.
* Transfer IP rights to another user (**Bob**).
* Verify IP ownership after transfer.
* Ensure **peer1.org1.example.com** is correctly synced with the network.

***

### **Support Commands**

#### **Set Environment for Org1 (Including peer1.org1.example.com)**

```bash
source ./scripts/setOrgPeerContext.sh 1

export FABRIC_CFG_PATH=${PWD}/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
echo "✅ Org1 environment setup complete!"

# Switch to Peer1
export CORE_PEER_ADDRESS=localhost:8051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
echo "✅ Org1 Peer1 environment setup complete!"
```

#### **Set Environment for Org2**

```bash
source ./scripts/setOrgPeerContext.sh 2
export FABRIC_CFG_PATH=${PWD}/configtx
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
echo "✅ Org2 environment setup complete!"
```

#### **Check Network Health**

```bash
peer lifecycle chaincode queryinstalled
peer channel list
peer channel getinfo -c ipchannel
```

If you get an access issue, use:

```bash
sudo chmod -R 755 /home/ubuntu/challenge/test-network/organizations/
```

#### **Joining New Peer to ipchannel**

```bash
export CORE_PEER_ADDRESS=localhost:8051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
peer channel join -b ./channel-artifacts/ipchannel.block
```

#### **Deploying Chaincode (ipcc) on ipchannel**

```bash
./network.sh deployCC -c ipchannel -ccn ipcc -ccp ../chaincode -ccl go
```

#### **Restarting the Network (If Needed)**

```bash
./network.sh down
```

This ensures your **Intellectual Property (IP) Rights Management System** is properly deployed!

***

### **Chaincode Execution Commands**

#### **Register an Intellectual Property (IP)**

**Function:** `RegisterIP(ipID, owner, title, registered)`

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C ipchannel -n ipcc --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["RegisterIP","ip1","Alice","Blockchain Patent","2025-03-06"]}'
```

#### **Retrieve Intellectual Property Ownership**

**Function:** `VerifyOwnership(ipID)`

```bash
peer chaincode query -C ipchannel -n ipcc -c '{"Args":["VerifyOwnership","ip1"]}'
```

#### **Transfer Ownership of IP**

**Function:** `TransferIP(ipID, newOwner)`

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C ipchannel -n ipcc --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["TransferIP","ip1","Bob"]}'
```

#### **Query Updated IP Ownership**

Verify that ownership transfer was successful.

```bash
peer chaincode query -C ipchannel -n ipcc -c '{"Args":["VerifyOwnership","ip1"]}'
```

***

This structured guide ensures a **clear, step-by-step** deployment and operation of the **decentralized IP Rights Management System** using **Hyperledger Fabric**.
