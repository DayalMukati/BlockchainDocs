# Question 2: Write and Deploy Certificate Issuance Chaincode

You are tasked with developing a **certificate issuance smart contract** for a **Hyperledger Fabric network**. The goal is to create a smart contract that **issues certificates, verifies them, and retrieves certificate details**.

#### **Required Chaincode Functions**

1. **IssueCertificate(certID, studentName, courseName)** → Issue a new certificate with a unique certificate ID, student name, and course name.
2. **VerifyCertificate(certID)** → Check if a given certificate exists on the blockchain.
3. **GetCertificate(certID)** → Retrieve the details of a specific certificate.

#### **Deployment Environment**

Use the provided **pre-configured test network**, which includes:

* **Two peers**, one for `Org1` and one for `Org2`.
* **A default channel** named `mychannel`.
* **A pre-initialized ledger with no data**.

#### **Test the Smart Contract** by performing the following actions:

**Issue a certificate** with:

* `certID`: `cert1`
* `studentName`: `John Doe`
* `courseName`: `Blockchain Basics`

**Verify the certificate** with `cert1` to check if it exists.

**Retrieve certificate details** to confirm that it belongs to `John Doe`.

***

#### **Deployment Steps**

**Deploy the Chaincode**

```bash
cd fabric-samples/test-network
./network.sh deployCC -c mychannel -ccn certcc -ccp ../chaincode -ccl go
```

**Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

**Execute Transactions**

&#x20;**1. Issue a Certificate**

```bash
peer chaincode invoke -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls --cafile $ORDERER_CA -C mychannel -n certcc \
  --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
  --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
  -c '{"Args":["IssueCertificate","cert1","John Doe","Blockchain Basics"]}'
```

**2. Verify the Certificate**

```bash
peer chaincode query -C mychannel -n certcc -c '{"Args":["VerifyCertificate","cert1"]}'
```

**3. Retrieve Certificate Details**

```bash
peer chaincode query -C mychannel -n certcc -c '{"Args":["GetCertificate","cert1"]}'
```

***

#### **Notes:**

The following steps are already performed for you as prerequisites:

The **Hyperledger Fabric test network** can be started using the command:

```bash
./network.sh up createChannel
```

with the default channel **mychannel**.

**Peers for Org1 and Org2** are already joined to the channel.

The **chaincode environment** setup is pre-configured in the **chaincode directory**.
