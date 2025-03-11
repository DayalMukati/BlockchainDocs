# Question 1: Write and Deploy Student Record Management System Chaincode

**Problem Statement**

Your organization is building a **decentralized Student Record Management System** using **Hyperledger Fabric**. The goal is to allow institutions to **register students, update course details, and retrieve student records** in a **trustless and immutable environment**.

In this task, you will:

* **Develop a smart contract** that manages student records.
* **Deploy it on a Hyperledger Fabric network**.
* **Test the chaincode** by registering students, updating courses, and retrieving student details.

***

### **Tasks to be Completed**

1. **Write chaincode** that supports the following functions:

* `RegisterStudent(studentID, name, age, course)` → Registers a new student.
* `UpdateCourse(studentID, newCourse)` → Updates the student’s enrolled course.
* `GetStudent(studentID)` → Retrieves student details.

2. **Deploy the chaincode (`studentcc`)** on a pre-configured Hyperledger Fabric test network.
3. **Test the smart contract**:

* Register a **student John Doe (ID: student1, Age: 22, Course: Blockchain Basics)**.
* Update John Doe’s course to **Hyperledger Fabric**.
* Retrieve and confirm the student details.

## **Deployment Steps**

**Deploy the Chaincode**

```
cd challange/test-network
sudo ./network.sh deployCC -c mychannel -ccn studentcc -ccp ../chaincode -ccl go
```

**Set Environment for Org1**

```
export FABRIC_CFG_PATH=${PWD}/configtx
source ./scripts/setOrgPeerContext.sh 1

```

### **Chaincode Execution Commands**

#### **1. Register a Student**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n studentcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterStudent", "student1", "John Doe", "22", "Blockchain Basics"]}'
```

***

#### **2. Retrieve Student Details**

```bash
peer chaincode query -C mychannel -n studentcc \
    -c '{"Args":["GetStudent", "student1"]}'
```

***

#### **3. Update Student Course**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n studentcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["UpdateCourse", "student1", "Hyperledger Fabric"]}'
```

***

#### **4. Confirm Course Update**

```bash
peer chaincode query -C mychannel -n studentcc \
    -c '{"Args":["GetStudent", "student1"]}'
```
