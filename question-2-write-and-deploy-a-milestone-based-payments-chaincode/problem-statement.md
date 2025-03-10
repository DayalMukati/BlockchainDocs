# Problem Statement

## **Problem Statement:**

You are tasked with writing a milestone-based payment smart contract for a Hyperledger Fabric network. The goal is to create a smart contract that manages payments tied to specific project or order milestones.

You are required to:

Write a smart contract that supports the following functionalities:

* **CreateMilestonePayment(orderID, milestoneID, amount):** Create a payment installment tied to a specific milestone.
* **UpdateMilestoneStatus(orderID, milestoneID, status):** Update the status of a milestone (e.g., "completed", "pending").
* **ReleaseMilestonePayment(orderID, milestoneID):** Release payment for a completed milestone.
* **QueryMilestonePayments(orderID):** Retrieve all milestone payments associated with an order.

Deploy this smart contract on a test network that is already set up with a default channel.

Use the provided pre-configured test network, which includes the following:

* Two peers, one for Org1 and one for Org2.
* A default channel named `mychannel`.
* A pre-initialized ledger with no data.

Test the smart contract by performing the following actions:

1. Create a milestone payment for an order with ID `order1`, milestone ID `milestone1`, and amount `1000`.
2. Update the milestone status to `completed`.
3. Release the payment for the completed milestone.
4. Retrieve and display all milestones and their statuses for `order1`.

#### **Requirements:**

* The smart contract should be written in Go (Golang).
* The test network should be used to test the contract, and test results should be displayed using `peer chaincode invoke` and `peer chaincode query`.

#### Notes:

The following steps are already performed for you as prerequisites:

* The Hyperledger Fabric test network can be started using the command `./network.sh up createChannel` with a default channel `mychannel`.
* The peers for `Org1` and `Org2` are joined to the channel.
* The chaincode environment setup is pre-configured in the `chaincode` directory.
*   Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn supplychain -ccp ../chaincode -ccl go
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
    --tls --cafile $ORDERER_CA -C mychannel -n milestonepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateMilestonePayment","order1","milestone1","1000"]}'
```

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n milestonepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["UpdateMilestoneStatus","order1","milestone1","completed"]}'
```

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n milestonepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ReleaseMilestonePayment","order1","milestone1"]}'
```

```
peer chaincode query -C mychannel -n milestonepayment -c '{"Args":["QueryMilestonePayments","order1"]}'
```

You only need to write the smart contract code and deploy it.
