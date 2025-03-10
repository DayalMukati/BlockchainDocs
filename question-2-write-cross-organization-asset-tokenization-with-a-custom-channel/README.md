# Question 2: Write Cross-Organization Asset Tokenization with a Custom Channel

Your company is building a **decentralized asset tokenization platform** using **Hyperledger Fabric**. The goal is to tokenize physical or digital assets, allowing **secure ownership transfer and asset management across multiple organizations**.

Your task is to:

**1. Create a new channel named `tokenizationchannel`**.\
**2. Configure and generate channel artifacts (`tokenizationchannel.tx`)**.\
**3. Join `Org1 (Issuer)` and `Org2 (Investor)` to the new channel**.\
**4. Deploy the Asset Tokenization Chaincode (`tokencc`) onto `tokenizationchannel`**.\
**5. Test transactions by minting tokens, transferring ownership, and burning tokens**.

***

### **Steps to be Performed**

#### **1. Create a New Channel**

* Modify `configtx.yaml` to define `tokenizationchannel`
* Use `configtxgen` to generate `tokenizationchannel.tx`
* Create the channel using `peer channel create`
* Join `Org1` and `Org2` to `tokenizationchannel`

#### **2. Deploy the Chaincode**

* Deploy **`tokencc`** onto `tokenizationchannel`
* Ensure endorsement policy allows both **Org1 and Org2** to validate transactions

#### **3. Smart Contract Implementation**

* Write a chaincode that:
  * **Mints new tokens** for an asset
  * **Transfers token ownership** between parties
  * **Burns tokens** when the asset is redeemed or destroyed
  * **Retrieves asset details and balances**

#### **4. Testing & Validation**

* **Mint new tokens for an asset**
* **Transfer tokens to another organization**
* **Burn tokens upon asset redemption**
* **Query asset balances and details**

***

### **Expected Transactions**

| **Transaction**     | **Args Example**                             |
| ------------------- | -------------------------------------------- |
| **MintTokens**      | `["MintTokens", "asset1", "100", "Org1"]`    |
| **TransferTokens**  | `["TransferTokens", "asset1", "50", "Org2"]` |
| **BurnTokens**      | `["BurnTokens", "asset1", "20"]`             |
| **GetAssetBalance** | `["GetAssetBalance", "Org1"]`                |



### **Support Commands for Setting Up Environment Variables (Org1 & Org2) in Tokenization Network**

These commands will **set up the correct environment** for interacting with **Org1 (Issuer)** and **Org2 (Investor)** in the **Hyperledger Fabric network**.

***

#### **1. Set Environment for Org1 (Issuer)**

```bash
# Set the context for Org1's peer
source ./scripts/setOrgPeerContext.sh 1

# Update the PATH to include Fabric binaries
export FABRIC_CFG_PATH=${PWD}/configtx

# Define Org1 TLS certificates and MSP configuration
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

```

***

#### **2. Set Environment for Org2 (Investor)**

```bash
# Set the context for Org2's peer
source ./scripts/setOrgPeerContext.sh 2

# Update the PATH to include Fabric binaries
export FABRIC_CFG_PATH=${PWD}/configtx

# Define Org2 TLS certificates and MSP configuration
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051

```

***

#### **3. Set Orderer Environment Variables**

```bash
# Define Orderer TLS and MSP settings
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key

```

***

#### **4. Check Peer Status**

Run these commands to check if **Org1 and Org2** are properly configured:

```bash
# Check if peers are active
peer lifecycle chaincode queryinstalled
peer channel list
peer channel getinfo -c tokenizationchannel
```

***

#### 5. Install chaincode

```
cd fabric-samples/test-network
sudo ./network.sh deployCC -c tokenizationchannel -ccn tokencc -ccp ../chaincode -ccl go
```



#### **4.** Mint New Tokens for an Asset

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C tokenizationchannel -n tokencc --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["MintTokens","asset1","1000","Org1"]}'

```

5. Retrieve Asset Balance

```
peer chaincode query -C tokenizationchannel -n tokencc -c '{"Args":["GetAssetBalance","asset1"]}'

```

6. Transfer Token Ownership

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C tokenizationchannel -n tokencc --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["TransferTokens","asset1","500","Org2"]}'

```

7. Burn Tokens (Remove Tokens)

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C tokenizationchannel -n tokencc --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["BurnTokens","asset1","500"]}'

```



8. **Restarting the Network (if needed)**

If you need to **restart the network**, use:

```bash
cd fabric-samples/test-network
./network.sh down
```
