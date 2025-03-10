# Question 3: Write and Deploy Simple Medical Records Chaincode

You are tasked with developing a **medical records smart contract** for a **Hyperledger Fabric network**. The goal is to create a smart contract that **registers patient records, updates medical history, and retrieves patient details**.

#### **Required Chaincode Functions**

1. **RegisterPatient(patientID, name, age, medicalHistory)** → Register a new patient with a unique patient ID, name, age, and initial medical history.
2. **UpdateMedicalHistory(patientID, newHistory)** → Append new medical history to the existing record.
3. **GetPatientDetails(patientID)** → Retrieve the details of a specific patient.

#### **Deployment Environment**

Use the provided **pre-configured test network**, which includes:

* **Two peers**, one for `Org1` and one for `Org2`.
* **A default channel** named `mychannel`.
* **A pre-initialized ledger with no data**.

#### **Test the Smart Contract** by performing the following actions:

**Register a patient** with:

* `patientID`: `patient1`
* `name`: `Alice Johnson`
* `age`: `30`
* `medicalHistory`: `"No known allergies"`

**Update the medical history** of `patient1` with `"Diagnosed with Hypertension"`.

**Retrieve patient details** to confirm that the **medical history now includes `"Diagnosed with Hypertension"`**.

***

#### **Deployment Steps**

**Deploy the Chaincode**

```bash
cd fabric-samples/test-network
sudo ./network.sh deployCC -c mychannel -ccn medicalcc -ccp ../chaincode -ccl go
```

**Set Environment for Org1**

```bash
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1
```

**Execute Transactions**

**1. Register a Patient**

```bash
peer chaincode invoke -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls --cafile $ORDERER_CA -C mychannel -n medicalcc \
  --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
  --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
  -c '{"Args":["RegisterPatient","patient1","Alice Johnson","30","No known allergies"]}'
```

**2. Update Medical History**

```bash
peer chaincode invoke -o localhost:7050 \
  --ordererTLSHostnameOverride orderer.example.com \
  --tls --cafile $ORDERER_CA -C mychannel -n medicalcc \
  --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
  --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
  -c '{"Args":["UpdateMedicalHistory","patient1","Diagnosed with Hypertension"]}'
```

**3. Retrieve Patient Details**

```bash
peer chaincode query -C mychannel -n medicalcc -c '{"Args":["GetPatientDetails","patient1"]}'
```
