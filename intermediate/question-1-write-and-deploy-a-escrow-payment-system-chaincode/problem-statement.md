# Problem Statement

### **Problem Statement:**

You are tasked with writing a smart contract for a simplified escrow payment system on a Hyperledger Fabric network. The goal is to hold funds in escrow until both the payer and payee approve the transaction, at which point the funds are released.

You are required to:

Write a smart contract that supports the following functionalities:

* **CreateEscrow(escrowID, payerID, payeeID, amount):** Create a new escrow account where funds are locked until both the payer and payee approve.
* **ApproveEscrow(escrowID, approverID):** Record an approval for the escrow by either the payer or payee.
* **ReleaseEscrow(escrowID):** Release the funds from escrow to the payee after both the payer and payee have approved the transaction.
* **QueryEscrowStatus(escrowID):** Retrieve the current status of an escrow account, including whether it has been approved and if the funds have been released.

### **Requirements:**

* The payer and payee must approve the escrow to release the funds.
* The contract should allow querying the escrow account to check if the funds are still in escrow or have been released.
* The smart contract should be written in Go (Golang).
* The test network should be used to test the contract, and test results should be displayed using `peer chaincode invoke` and `peer chaincode query`.

### **Test the smart contract by performing the following actions:**

* **Create an escrow account** with ID `escrow1`, where the payer is `payer1`, the payee is `payee1`, and the amount is `1000`.
* **Approve the escrow as the payer** (`payer1`), which indicates that the payer is ready for the transaction to proceed.
* **Confirm the escrow as the payee** (`payee1`), which signals the payee's approval to receive the funds.
* **Release the escrow funds** for `escrow1` to `payee1` after both parties approve.
* **Query the escrow status** for `escrow1` to verify that the funds have been released.

#### Notes:

The following steps are already performed for you as prerequisites:

* The Hyperledger Fabric test network can be started using the command `./network.sh up createChannel` with a default channel `mychannel`.
* The peers for `Org1` and `Org2` are joined to the channel.
* The chaincode environment setup is pre-configured in the `chaincode` directory.
*   Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn escrowpayment -ccp ../chaincode -ccl go
    ```
*   **A script is provided in the `scripts` folder to set the environment variables for Org1 and Org2 peers. You can use the following command to set the context for Org1:**

    ```bash
    source ./scripts/setOrgPeerContext.sh 1
    ```

#### Support Commands:&#x20;

<pre><code>source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
<strong>export FABRIC_CFG_PATH=${PWD}/configtx
</strong></code></pre>

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateEscrow","escrow1","payer1","payee1","1000"]}'
```

```sh
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApproveEscrow","escrow1","payer1"]}'
```

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ReleaseEscrow","escrow1"]}'
```

```
peer chaincode query -C mychannel -n escrowpayment -c '{"Args":["QueryEscrowStatus","escrow1"]}'
```

You only need to write the smart contract code and deploy it.
