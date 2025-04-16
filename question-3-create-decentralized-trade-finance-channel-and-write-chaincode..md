# Question 3: Create Decentralized Trade Finance Channel and Write Chaincode.

Your organization is building a **Decentralized Trade Finance Platform** using **Hyperledger Fabric**. The goal is to allow **importers, exporters, and banks to handle Letters of Credit (LoC) in a trustless and transparent manner**.

To ensure **security and efficiency**, you must:

* **Create a new channel (`tradechannel`)** to handle trade finance transactions.
* **Deploy the Trade Finance Chaincode (`tradecc`)** that enables **importers to request LoC, banks to approve LoC, and exporters to complete shipments**.
* **Ensure that only authorized banks can approve Letters of Credit**.

This challenge requires **advanced Hyperledger Fabric network setup**, including **creating a new channel, adding peers, and deploying smart contracts**.

***

### **Tasks to be Completed**

#### **1. Create a New Channel (`tradechannel`)**

* Define a **secure channel** for trade finance transactions.
* Ensure **both Org1 (Importer) and Org2 (Exporter) join the channel**.

#### **2. Deploy the Trade Finance Chaincode (`tradecc`)**

* Implement **functions** to **request LoC, approve LoC, complete shipments, and verify transactions**.
* Ensure that **only authorized banks can approve the LoC**.

#### **3. Test Smart Contract Functionality**

* **Importer requests a Letter of Credit for $100,000**.
* **Bank approves the LoC**.
* **Exporter ships the goods and submits proof**.
* **LoC is marked as settled once the shipment is verified**.

***

### **Deployment Steps**

#### **1. Deploy the Chaincode (`tradecc`)**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c tradechannel -ccn tradecc -ccp ../chaincode -ccl go
```

***

#### **2. Set Environment for Org1 (Importer) & Org2 (Exporter)**

```bash
source ./scripts/setOrgPeerContext.sh 1
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
echo "Importer environment setup complete!"

source ./scripts/setOrgPeerContext.sh 2
export CORE_PEER_ADDRESS=localhost:9051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
echo "Exporter environment setup complete!"
```

#### **3. Request a Letter of Credit (LoC)**

**Importer requests a LoC for $100,000 from ImporterA to ExporterB.**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C tradechannel -n tradecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RequestLoC", "loc1", "ImporterA", "ExporterB", "100000"]}'
```

***

#### **4.  Approve the Letter of Credit**

**Bank (BankX) approves the LoC.**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C tradechannel -n tradecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApproveLoC", "loc1", "BankX"]}'
```

***

#### **5. Check LoC Status**

**Verify if the LoC has been approved.**

```bash
peer chaincode query -C tradechannel -n tradecc -c '{"Args":["GetLoC","loc1"]}'
```

***

#### **6. Mark Shipment as Completed**

**Exporter marks the shipment as completed.**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C tradechannel -n tradecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CompleteShipment", "loc1"]}'
```

***

#### **7. Check Shipment Status**

**Verify if the shipment is marked as completed.**

```bash
peer chaincode query -C tradechannel -n tradecc -c '{"Args":["GetLoC","loc1"]}'
```

***

#### **8. Settle the LoC**

**Mark the LoC as settled after shipment verification.**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C tradechannel -n tradecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["SettleLoC", "loc1"]}'
```

***

#### **9. Verify Final Settlement**

**Check if the LoC is settled successfully.**

```bash
peer chaincode query -C tradechannel -n tradecc -c '{"Args":["GetLoC","loc1"]}'
```
