# Question 3: Write and Deploy Smart Invoice Management Chaincode

Your organization is building a **Smart Invoice Management System** using **Hyperledger Fabric**. The goal is to **automate invoicing, allow real-time tracking, approve payments, and settle invoices securely** between buyers and sellers.

In this task, you will:

* **Develop a smart contract** that enables **invoice creation, approval, settlement, and dispute resolution**.
* **Deploy it on a Hyperledger Fabric network**.
* **Test the chaincode** by **creating an invoice, approving it, making the payment, and checking invoice status**.

***

### **Tasks to be Completed**

1. **Write a chaincode** that supports the following functions:

* `CreateInvoice(invoiceID, buyer, seller, amount, dueDate, status)` → Seller generates an invoice for a buyer.
* `ApproveInvoice(invoiceID)` → Buyer approves the invoice for payment.
* `SettleInvoice(invoiceID)` → The invoice is paid, and funds are transferred.
* `DisputeInvoice(invoiceID, reason)` → Buyer disputes the invoice due to discrepancies.
* `GetInvoiceDetails(invoiceID)` → Retrieve invoice details for verification.

2. **Deploy the chaincode (`invoicecc`)** on a **pre-configured test network**.
3. **Test the smart contract**:

* **Seller (Bob) generates an invoice of $5000 for Buyer (Alice).**
* **Buyer (Alice) approves the invoice.**
* **Invoice is settled successfully.**
* **Retrieve invoice details to verify the payment.**

***

### **Deployment Steps**

#### **1. Deploy the Chaincode**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c mychannel -ccn invoicecc -ccp ../chaincode -ccl go
```

***

#### **2. Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

### **Chaincode Execution Commands**

#### **3. Create an Invoice**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n invoicecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateInvoice", "inv1", "Alice", "Bob", "5000", "2025-04-01"]}'
```

***

#### **4. Approve the Invoice**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n invoicecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApproveInvoice", "inv1"]}'
```

***

#### **5. Settle the Invoice**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n invoicecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["SettleInvoice", "inv1"]}'
```

***

#### **6. Retrieve Invoice Details**

```bash
peer chaincode query -C mychannel -n invoicecc \
    -c '{"Args":["GetInvoiceDetails", "inv1"]}'
```
