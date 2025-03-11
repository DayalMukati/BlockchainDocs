# Question 2: Write and Deploy Peer-to-Peer Payment Chaincode

Your organization is developing a **decentralized Peer-to-Peer (P2P) Payment System** using **Hyperledger Fabric**. The goal is to allow users to **register accounts, transfer funds, and check balances** securely.

In this task, you will:

* **Develop a smart contract** that enables **secure money transfers** between users.
* **Deploy it on a Hyperledger Fabric network**.
* **Test the chaincode** by creating accounts, transferring funds, and verifying balances.

***

### **Tasks to be Completed**

1. **Write a chaincode** that supports the following functions:

* `RegisterAccount(accountID, name, balance)` → Creates a new account.
* `TransferFunds(fromAccount, toAccount, amount)` → Transfers funds securely.
* `GetBalance(accountID)` → Retrieves the account balance.

2. **Deploy the chaincode (`paymentscc`)** on a **pre-configured test network**.
3. **Test the smart contract**:

* Register two accounts:
  * **Alice (account1, balance: 1000)**
  * **Bob (account2, balance: 500)**
* Transfer **200 tokens from Alice to Bob**.
* Retrieve and verify **Alice’s and Bob’s balances**.

***

### **Deployment Steps**

#### **1. Deploy the Chaincode**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c mychannel -ccn paymentscc -ccp ../chaincode -ccl go
```

***

#### **2. Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

***

### **Chaincode Execution Commands**

#### **3. Register Alice's Account**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n paymentscc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterAccount", "account1", "Alice", "1000"]}'
```

***

#### **4. Register Bob's Account**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n paymentscc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterAccount", "account2", "Bob", "500"]}'
```

***

#### **5. Transfer 200 Tokens from Alice to Bob**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n paymentscc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["TransferFunds", "account1", "account2", "200"]}'
```

***

#### **6. Check Alice’s Balance**

```bash
peer chaincode query -C mychannel -n paymentscc \
    -c '{"Args":["GetBalance", "account1"]}'
```

***

#### **7. Check Bob’s Balance**

```bash
peer chaincode query -C mychannel -n paymentscc \
    -c '{"Args":["GetBalance", "account2"]}'
```
