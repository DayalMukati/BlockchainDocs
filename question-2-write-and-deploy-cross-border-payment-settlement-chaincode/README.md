# Question 2: write and Deploy Cross-Border Payment Settlement Chaincode

Your organization is building a **Cross-Border Payment Settlement System** using **Hyperledger Fabric**. The goal is to **enable secure, transparent, and instant settlements between banks and financial institutions** across different countries.

In this task, you will:

* **Develop a smart contract** that enables **payment initiation, approval, settlement, and refund processing**.
* **Deploy it on a Hyperledger Fabric network**.
* **Test the chaincode** by **initiating a payment, approving it, settling the amount, and processing a refund if required**.

***

### **Tasks to be Completed**

1. **Write a chaincode** that supports the following functions:

* `InitiatePayment(paymentID, senderBank, receiverBank, amount, status)` → A sender bank initiates a payment request.
* `ApprovePayment(paymentID)` → The receiver bank approves the payment.
* `SettlePayment(paymentID)` → The payment gets settled and funds are transferred.
* `RefundPayment(paymentID)` → If a payment is disputed, the sender receives a refund.
* `GetPaymentDetails(paymentID)` → Retrieve payment details for verification.

2. **Deploy the chaincode (`paymentscc`)** on a **pre-configured test network**.
3. **Test the smart contract**:

* **Bank A (sender) initiates a payment of $5000 to Bank B (receiver).**
* **Bank B approves the payment.**
* **Payment is settled successfully.**
* **Retrieve the payment details to verify the transaction.**

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

#### **3. Initiate Payment**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n paymentscc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["InitiatePayment", "payment1", "BankA", "BankB", "5000"]}'
```

***

#### **4. Approve the Payment**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n paymentscc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApprovePayment", "payment1"]}'
```

***

#### **5. Settle the Payment**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n paymentscc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["SettlePayment", "payment1"]}'
```

***

#### **6. Retrieve Payment Details**

```bash
peer chaincode query -C mychannel -n paymentscc \
    -c '{"Args":["GetPaymentDetails", "payment1"]}'
```

