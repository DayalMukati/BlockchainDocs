# Question 3: Adding a New Peer to Org1 & Deploying Intellectual Property Chaincode

### **Problem Statement: Add a New Peer & Deploy IP Rights Chaincode**

Your organization is building a **decentralized Intellectual Property (IP) rights management system** using **Hyperledger Fabric**. The goal is to allow **creators to register IP, transfer rights, and verify ownership** in an immutable, trustless system.

To scale this system, you must **add a new peer (`peer1.org1.example.com`) to Org1**, ensuring **high availability and redundancy**. Once the new peer is added, you will **deploy the IP Rights Management Chaincode (`ipcc`)** on a new channel called `ipchannel`.

***

### **Tasks to be Completed**

**1. Add a new peer (`peer1.org1.example.com`) to Org1.**\
**2. Configure Org1’s MSP and update `docker-compose` settings for the new peer.**\
**3. Start the new peer and join it to `ipchannel`.**\
**4. Deploy the Intellectual Property Chaincode (`ipcc`) on `ipchannel`.**\
**5. Test transactions by registering an IP, transferring ownership, and verifying ownership.**



#### **Steps to be Performed**

**1. Add a New Peer (`peer1.org1.example.com`) to Org1**

* Modify `crypto-config.yaml` to define the new peer.
* Use `cryptogen` to generate crypto materials for the new peer.
* Update `docker-compose` files to include `peer1.org1.example.com`.
* Start the new peer using `docker-compose up -d`.
* Join `peer1.org1.example.com` to `ipchannel`.

**2. Deploy the Chaincode**

* Deploy **`ipcc`** onto `ipchannel`.
* Ensure endorsement policy includes **both Org1 peers (`peer0` and `peer1`)** and Org2.

**3. Smart Contract Implementation**

Write a chaincode that:

* Registers a new Intellectual Property (IP).
* Transfers ownership of IP between parties.
* Verifies IP ownership.
* Retrieves IP details.

**4. Testing & Validation**

* **Register an IP** for a creator.
* **Transfer IP rights** to another organization.
* **Verify IP ownership** after the transfer.
* **Ensure `peer1.org1.example.com` is correctly synced with the network**.

#### **Support Commands**

**1. Set Environment for Org1 (Including `peer1.org1.example.com`)**

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

**2. Set Environment for Org2**

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

**3. Check Network Health**

```bash
peer lifecycle chaincode queryinstalled
peer channel list
peer channel getinfo -c ipchannel
```

**4. Joining New Peer to `ipchannel`**

```bash
export CORE_PEER_ADDRESS=localhost:8051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
peer channel join -b ./channel-artifacts/ipchannel.block
```

**5. Approving and Committing Chaincode**

```bash
./network.sh deployCC -c ipchannel -ccn ipcc -ccp ../chaincode -ccl go
```

**6. Restarting the Network (If Needed)**

```bash
cd fabric-samples/test-network
./network.sh down
```
