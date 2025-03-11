# Question 3: Write and Deploy Loan Management Chaincode

Your organization is building a **decentralized Loan Management System** using **Hyperledger Fabric**. The goal is to allow users to **apply for loans, approve loans, and repay them** while ensuring the **immutability and security** of transactions.

In this task, you will:

* **Develop a smart contract** that enables **secure loan processing** between borrowers and lenders.
* **Deploy it on a Hyperledger Fabric network**.
* **Test the chaincode** by applying for a loan, approving the loan, and making a repayment.

***

### **Tasks to be Completed**

1. **Write a chaincode** that supports the following functions:

* `ApplyForLoan(loanID, borrowerID, amount, status)` → A borrower applies for a loan.
* `ApproveLoan(loanID, lenderID)` → A lender approves the loan request.
* `RepayLoan(loanID, amount)` → The borrower repays part of the loan.

2. **Deploy the chaincode (`loancc`)** on a **pre-configured test network**.
3. **Test the smart contract**:

* Borrower (Alice) applies for a loan of $1000.
* Lender (Bank1) approves the loan.
* Alice repays $500.
* Check outstanding balance.

***

### **Deployment Steps**

#### **1. Deploy the Chaincode**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c mychannel -ccn loancc -ccp ../chaincode -ccl go
```

***

#### **2. Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

***

### **Chaincode Execution Commands**

#### **3. Apply for a Loan**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n loancc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApplyForLoan", "loan1", "Alice", "1000"]}'
```

***

#### **4. Approve the Loan**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n loancc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApproveLoan", "loan1"]}'
```

***

#### **5. Repay $500**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n loancc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RepayLoan", "loan1", "500"]}'
```

***

#### **6. Check Loan Details**

```bash
peer chaincode query -C mychannel -n loancc \
    -c '{"Args":["GetLoan", "loan1"]}'
```
