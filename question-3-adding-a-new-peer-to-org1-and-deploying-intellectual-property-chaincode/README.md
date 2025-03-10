# Question 3: Adding a New Peer to Org1 & Deploying Intellectual Property Chaincode

Your organization is building a **decentralized Intellectual Property (IP) rights management system** using **Hyperledger Fabric**. The goal is to allow creators to **register IP, transfer rights, and verify ownership** in an **immutable, trustless system**.

To enhance **scalability and redundancy**, you need to:

* **Add a new peer (`peer1.org1.example.com`)** to Org1.
* **Create a new custom channel (`ipchannel`)** to manage IP transactions separately.
* **Deploy the IP Rights Management Chaincode (`ipcc`)** on the new channel.
* **Ensure that both Org1 and Org2 can participate** in managing IP ownership.

***

### **Tasks to be Completed**

1. **Add a new peer (`peer1.org1.example.com`)** to Org1 and it should be running on 8051 port.
2. **Configure Org1’s MSP and update docker-compose settings** for the new peer.
3. **Start the new peer and join it to `ipchannel`**.
4. **Deploy the Intellectual Property Chaincode (`ipcc`)** on `ipchannel`.
5. **Test transactions** by registering an IP, transferring ownership, and verifying ownership.

***

### **Steps to be Performed**

#### **1. Add a New Peer (`peer1.org1.example.com`) to Org1**

* Modify **`crypto-config.yaml`** to define the new peer.
* Use **`cryptogen`** to generate crypto materials for the new peer.
* Update **docker-compose files** to include `peer1.org1.example.com`.
* Start the **new peer** using `docker-compose up -d`.
* Join `peer1.org1.example.com` to `ipchannel`.

#### **2. Deploy the Chaincode**

* Deploy **`ipcc`** onto `ipchannel`.
* Ensure **endorsement policy** includes **both Org1 peers (`peer0` & `peer1`) and Org2**.

#### **3. Smart Contract Implementation**

The chaincode should support:

* **Registering an Intellectual Property (IP).**
* **Transferring ownership of IP between parties.**
* **Verifying IP ownership.**
* **Retrieving IP details.**

#### **4. Testing & Validation**

* **Register an IP for a creator**.
* **Transfer IP rights** to another organization.
* **Verify IP ownership after transfer**.
* **Ensure `peer1.org1.example.com` is correctly synced** with the network.

***

### **Support Commands**

#### **1. Set Environment for Org1 (Including `peer1.org1.example.com`)**

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

***

#### **2. Set Environment for Org2**

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

***

#### **3. Check Network Health**

```bash
peer lifecycle chaincode queryinstalled
peer channel list
peer channel getinfo -c ipchannel
```

***

if you get the access issue then use below cmd or `sudo`:

```
sudo chmod -R 755 /home/ubuntu/challenge/test-network/organizations/
```

#### **4. Joining New Peer to `ipchannel`**

```bash
export CORE_PEER_ADDRESS=localhost:8051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
peer channel join -b ./channel-artifacts/ipchannel.block
```

***

#### **5️⃣ Deploying Chaincode (`ipcc`) on `ipchannel`**

```bash
./network.sh deployCC -c ipchannel -ccn ipcc -ccp ../chaincode -ccl go
```

***

#### **6️⃣ Restarting the Network (If Needed)**

```bash
./network.sh down
```

**This ensures your Intellectual Property (IP) Rights Management system is properly deployed!**&#x20;
