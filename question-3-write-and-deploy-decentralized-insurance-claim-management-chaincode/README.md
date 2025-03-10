# Question 3: Write and Deploy Decentralized Insurance Claim Management Chaincode

**Blockchain-Based Insurance Claim Processing System using Hyperledger Fabric**

You are tasked with implementing a blockchain-based insurance claim processing system using Hyperledger Fabric. The goal is to allow policyholders to file insurance claims, have them reviewed by an insurer, and process the payout.

***

#### **Smart Contract Functionalities**

Your smart contract should support the following functionalities:

* **Register a policyholder (`RegisterPolicyholder`)**: A new policyholder can be registered with a policy ID and initial balance.
* **File an insurance claim (`FileClaim`)**: A policyholder can file a claim, specifying claim ID, policy ID, claim amount, and claim reason.
* **Approve or Reject a claim (`ReviewClaim`)**: The insurer reviews the claim and either approves or rejects it. If approved, the claim amount is paid out to the policyholder.
* **Get claim details (`GetClaimDetails`)**: Retrieve the details of a specific claim.
* **Get policyholder details (`GetPolicyholderDetails`)**: Retrieve a policyholder's details.

***

#### **Test the Smart Contract**

#### **Perform the Following Actions:**

1. **Register a policyholder (`policyholder1`)** with policy ID `P123` and a balance of `5000` tokens.
2. **File an insurance claim (`claim1`)** under policy `P123` for `1000` tokens, reason: _"Car accident"_ by `policyholder1`.
3. **Review the claim (`claim1`)** and approve it.
4. **Verify that `policyholder1` received the payout** and check the claim status.

***

#### **Requirements**

* The chaincode should be written in **Go (Golang)**.
* The **test network** should be used to test the contract.
* Test results should be displayed using `peer chaincode invoke` and `peer chaincode query`.

***

#### **Deployment Steps**

#### **Deploy the Chaincode**

```bash
cd test-network
sudo ./network.sh deployCC -c mychannel -ccn insurancecc -ccp ../chaincode -ccl go
```

#### **Set Environment for Org1**

```bash
source ./scripts/setOrgPeerContext.sh 1
export FABRIC_CFG_PATH=${PWD}/configtx
```

***

#### **Execute Transactions**

#### **1. Register Users (Policyholder)**

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n insurancecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["RegisterPolicyholder", "P123", "Alice", "5000"]}'
```

#### **2. File an Insurance Claim**

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n insurancecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["FileClaim", "claim1", "P123", "1000", "Car accident"]}'
```

#### **3. Approve a Claim**

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n insurancecc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ReviewClaim", "claim1", "true"]}'
```

#### **4. Retrieve Claim Status**

```bash
peer chaincode query -C mychannel -n insurancecc -c '{"Args":["GetClaimDetails","claim1"]}'
```
