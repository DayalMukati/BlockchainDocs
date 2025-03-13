# Question 1 : Create new Channel and Write Interbank Cross-Border Payment Settlement Chaincode

Your organization is developing a **Cross-Border Payment Settlement System** using **Hyperledger Fabric** to facilitate **secure and efficient interbank payments between multiple financial institutions**.

To enhance scalability and security, you need to:

* **Create a new channel (`paymentschannel`)** to handle secure interbank transactions.
* **Deploy the Cross-Border Payment Chaincode (`paymentcc`)** that allows banks to initiate transactions, verify payments, and settle funds.
* **Implement secure, multi-party settlement verification** before the transaction is finalized.

This challenge requires **advanced network configuration**, including **adding a new channel, joining peers, and deploying smart contracts**.

***

### **Tasks to be Completed**

#### **1. Create a New Channel (`paymentschannel`)**

* Define a new **channel (`paymentschannel`)** for secure interbank settlements.
* Ensure **both Org1 (Bank A) and Org2 (Bank B) join the channel**.

#### **2. Deploy the Cross-Border Payment Chaincode (`paymentcc`)**

* Implement **functions** to **initiate transactions, verify settlements, approve transactions, and finalize payments**.
* Ensure that **both banks must approve** before the transaction is settled.

#### **3. Test Smart Contract Functionality**

* **Bank A initiates a cross-border transaction of $10,000 to Bank B**.
* **Bank B approves the transaction**.
* **Funds are settled upon successful multi-party validation**.
* **Retrieve transaction details to verify completion**.

***

### **Deployment Steps**

#### **1. Deploy the Chaincode (`paymentcc`)**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c paymentschannel -ccn paymentcc -ccp ../chaincode -ccl go
```

2. **Set Environment for Org1 (Bank A) & Org2 (Bank B)**

```bash
source ./scripts/setOrgPeerContext.sh 1
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
echo "Bank A environment setup complete!"

source ./scripts/setOrgPeerContext.sh 2
export CORE_PEER_ADDRESS=localhost:9051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
echo "Bank B environment setup complete!"
```

### **Chaincode Execution Commands**

#### **3. Initiate Cross-Border Payment**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C paymentschannel -n paymentcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["InitiatePayment", "tx1", "BankA", "BankB", "10000"]}'
```

***

#### **4. Approve the Payment**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C paymentschannel -n paymentcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApprovePayment", "tx1"]}'
```

***

#### **5. Settle the Payment**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C paymentschannel -n paymentcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["SettlePayment", "tx1"]}'
```
