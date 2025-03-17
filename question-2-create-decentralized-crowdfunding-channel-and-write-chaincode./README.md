# Question 2: Create Decentralized Crowdfunding Channel and Write Chaincode.

Your organization is developing a **Decentralized Crowdfunding Platform** using **Hyperledger Fabric**. The goal is to allow **project creators to launch crowdfunding campaigns, backers to contribute funds, and campaigns to be finalized when funding goals are met**.

To ensure **security and trust**, you must:

* **Create a new channel (`crowdfundchannel`)** to handle all crowdfunding transactions.
* **Deploy the Crowdfunding Chaincode (`crowdfundcc`)** that enables project creation, funding, and withdrawal of funds when goals are met.
* **Ensure that only the project creator can withdraw funds after reaching the funding goal**.

This challenge requires **advanced Hyperledger Fabric network setup**, including **creating a new channel, adding peers, and deploying smart contracts**.

***

### **Tasks to be Completed**

#### **1. Create a New Channel (`crowdfundchannel`)**

* Define a **secure channel** for crowdfunding transactions.
* Ensure **both Org1 (Project Creators) and Org2 (Backers/Investors) join the channel**.

#### **2. Deploy the Crowdfunding Chaincode (`crowdfundcc`)**

* Implement **functions** to **create campaigns, contribute funds, and withdraw funds once the goal is met**.
* Ensure that **only the project creator can withdraw funds**.

#### **3. Test Smart Contract Functionality**

* **Project creator launches a crowdfunding campaign for $10,000**.
* **Backers contribute funds to the campaign**.
* **Verify if the campaign reaches its funding goal**.
* **Allow the project creator to withdraw the funds after the goal is met**.

***

### Deployment Steps

#### **1. Deploy the Chaincode (`crowdfundcc`)**

```bash
cd challenge/test-network
sudo ./network.sh deployCC -c crowdfundchannel -ccn crowdfundcc -ccp ../chaincode -ccl go
```

***

#### **2. Set Environment for Org1 (Project Creators) & Org2 (Backers/Investors)**

```bash
source ./scripts/setOrgPeerContext.sh 1
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
echo "Project Creator environment setup complete!"

source ./scripts/setOrgPeerContext.sh 2
export CORE_PEER_ADDRESS=localhost:9051
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
echo "Backers/Investors environment setup complete!"
```

#### **3. Create a Campaign**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C crowdfundchannel -n crowdfundcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    -c '{"Args":["CreateCampaign", "camp1", "CreatorA", "10000"]}'
```

***

#### **4. Contribute to the Campaign**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C crowdfundchannel -n crowdfundcc \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ContributeFunds", "camp1", "5000"]}'
```

***

#### **5. Withdraw Funds**

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C crowdfundchannel -n crowdfundcc \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    -c '{"Args":["WithdrawFunds", "camp1"]}'
```
