# Question 1: Write and Deploy Credit Scoring Chaincode

Your organization is developing a **Credit Scoring & Loan Risk Management System** using **Hyperledger Fabric**. The goal is to **evaluate borrowers' creditworthiness**, issue loans based on their credit score, and track repayments.

In this task, you will:

* **Develop a smart contract** that enables **secure credit scoring, loan issuance, and repayments**.
* **Deploy it on a Hyperledger Fabric network**.
* **Test the chaincode** by adding users, updating credit scores, issuing loans, making repayments, and verifying credit history.

***

### **Tasks to be Completed**

1. **Write a chaincode** that supports the following functions:

* `RegisterUser(userID, name, initialCreditScore)` → Registers a user with a starting credit score.
* `UpdateCreditScore(userID, newScore)` → Updates the user's credit score.
* `IssueLoan(loanID, userID, amount, status)` → Issues a loan if the credit score is sufficient.
* `RepayLoan(loanID, amount)` → Allows a borrower to make repayments.
* `GetUserCreditHistory(userID)` → Retrieves the user's past loan history.

2. **Deploy the chaincode (`creditscorecc`)** on a **pre-configured test network**.
3. **Test the smart contract**:

* Register a borrower **Alice (user1) with an initial credit score of 750**.
* Update **Alice’s credit score to 800**.
* Issue a loan of **$2000 to Alice** (since her credit score is above 700).
* Alice repays **$500**.
* Retrieve Alice's **credit history** to verify all transactions.

***

### **Deployment Steps**

#### **1. Deploy the Chaincode**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c mychannel -ccn creditscorecc -ccp ../chaincode -ccl go
```

***

#### **2. Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

***

### &#x20;**Chaincode Execution Commands**

#### **3. Register Alice as a User with an Initial Credit Score of 750**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n creditscorecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterUser", "user1", "Alice", "750"]}'
```

***

#### **4. Update Alice’s Credit Score to 800**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n creditscorecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["UpdateCreditScore", "user1", "800"]}'
```

***

#### **5. Issue a Loan of $2000 to Alice**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n creditscorecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["IssueLoan", "loan1", "user1", "2000"]}'
```
