# Question 1: Create Custom Identity Channel & DID Management Chaincod

Your company is building a **decentralized identity (DID) system** using **Hyperledger Fabric**. This system will allow users to create **self-sovereign identities**, update credentials, and verify identities **without relying on a central authority**.

\


Your task is to:

**1. Create a new channel named `identitychannel`**.

**2. Configure and generate channel artifacts (`identitychannel.tx`)**.

**3. Join `Org1 (Identity Provider)` and `Org2 (Verifier)` to the new channel**.

**4. Deploy the Digital Identity Management Chaincode (`identitycc`) onto `identitychannel`**.

**5. Test transactions by creating a DID, updating credentials, and verifying identity**.

***

\


**Steps to be Performed**

**1. Create a New Channel**

* Modify `configtx.yaml` to define `identitychannel`
* Use `configtxgen` to generate `identitychannel.tx`
* Create the channel using `peer channel create`
* Join `Org1` and `Org2` to `identitychannel`

\


**2. Deploy the Chaincode**

* Deploy **`identitycc`** onto `identitychannel`
* Ensure endorsement policy allows both **Org1 and Org2** to validate transactions

\


**3. Smart Contract Implementation**

* Write a chaincode that:
  * **Creates a DID** for users
  * **Updates credentials** for a DID
  * **Verifies identity** upon approval
  * **Retrieves user identity details**

**4. Testing & Validation**

* **Create a DID** for a user
* **Update credentials** for the DID
* **Verify the DID**
* **Query identity details**

***

#### &#x20; <a href="#expected-transactions" id="expected-transactions"></a>

\


**Expected Transactions**

**Transaction**

**Args Example**

**CreateDID**

`["CreateDID", "did:user1", "Alice Johnson", "Passport Verified"]`

**UpdateCredentials**

`["UpdateCredentials", "did:user1", "Passport + Driving License Verified"]`

**VerifyDID**

`["VerifyDID", "did:user1"]`

**GetDID**

`["GetDID", "did:user1"]`

***

\


\


**Deliverables**

* **A functional new channel (`identitychannel`)**
* **Properly joined Org1 and Org2 to `identitychannel`**
* **Successfully deployed `identitycc` chaincode**
* **Correct execution of transactions (DID creation, update, and verification)**

\


**Support Commands for Setting Up Environment Variables (Org1 & Org2)**

These commands will **set up the correct environment** for interacting with **Org1** and **Org2** in the Hyperledger Fabric network.\


\


**1. Set Environment for Org1 (Identity Provider)**

```
# Set the context for Org1's peer
source ./scripts/setOrgPeerContext.sh 1

# Update the PATH to include Fabric binaries
export FABRIC_CFG_PATH=${PWD}/configtx

# Define Org1 TLS certificates and MSP configuration
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051

```

***

\


\


**2. Set Environment for Org2 (Verifier)**

```
# Set the context for Org2's peer
source ./scripts/setOrgPeerContext.sh 2

# Update the PATH to include Fabric binaries
export FABRIC_CFG_PATH=${PWD}/configtx

# Define Org2 TLS certificates and MSP configuration
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

***

**3. Set Orderer Environment Variables**

```
# Define Orderer TLS and MSP settings
export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
```

***

**4. Check Peer Status**

Run these commands to check if **Org1 and Org2** are properly configured:

```
peer lifecycle chaincode queryinstalled
peer channel list
peer channel getinfo -c identitychannel
```



**`5. Deploy Chaincode`**

\
`sudo ./network.sh deployCC -c identitychannel -ccn identitycc -ccp ../chaincode -ccl go`

\


**`6 Create DID`**

`peer chaincode invoke -o localhost:7050 \`\
&#x20;   `--ordererTLSHostnameOverride orderer.example.com \`\
&#x20;   `--tls --cafile $ORDERER_CA -C identitychannel -n identitycc \`\
&#x20;   `--peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \`\
&#x20;   `--peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \`\
&#x20;   `-c '{"Args":["CreateDID","did:user1","John Doe","Passport Issued"]}'`\
7\. Update DID

\


7. peer chaincode invoke -o localhost:7050 \\\
   &#x20;   \--ordererTLSHostnameOverride orderer.example.com \\\
   &#x20;   \--tls --cafile $ORDERER\_CA -C identitychannel -n identitycc \\\
   &#x20;   \--peerAddresses localhost:7051 --tlsRootCertFiles $PEER0\_ORG1\_CA \\\
   &#x20;   \--peerAddresses localhost:9051 --tlsRootCertFiles $PEER0\_ORG2\_CA \\\
   &#x20;   -c '{"Args":\["UpdateCredentials","did:user1","Passport + Driving License Verified"]}'



8. **Verifiy DID**

peer chaincode invoke -o localhost:7050 \\\
&#x20;   \--ordererTLSHostnameOverride orderer.example.com \\\
&#x20;   \--tls --cafile $ORDERER\_CA -C identitychannel -n identitycc \\\
&#x20;   \--peerAddresses localhost:7051 --tlsRootCertFiles $PEER0\_ORG1\_CA \\\
&#x20;   \--peerAddresses localhost:9051 --tlsRootCertFiles $PEER0\_ORG2\_CA \\\
&#x20;   -c '{"Args":\["VerifyDID","did:user1"]}'\
