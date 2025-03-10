# Problem Statement

#### **Problem Statement:**

In this expert-level task, you are required to complete two major parts:

1.  **Part 1: Add a New Peer to Org1 in the Existing Test Network**

    You need to add a new peer (`peer1.org1.example.com`) to Org1 in the existing Hyperledger Fabric test network. The peer should be fully operational, and it must be able to join the existing channel `mychannel` that already has `peer0.org1.example.com` as a member.
2.  **Part 2: Write a Smart Contract for Transaction Logging**

    You are tasked with developing a smart contract for a **Transaction Logging System**. The smart contract should log transactions between users and allow for querying past transactions. The contract must support the following functionalities:

    * **LogTransaction(transactionID, sender, receiver, amount):** Log a new transaction with the provided details.
    * **QueryTransaction(transactionID):** Retrieve the details of a specific transaction using its ID.
    * **QueryAllTransactions():** Retrieve all the transactions logged in the system.

#### **Requirements:**

* For **Part 1**, after adding the new peer to Org1 and joining it to `mychannel`, you need to demonstrate that the new peer can install and instantiate the smart contract that you write in **Part 2**.
* For **Part 2**, implement the smart contract in Go (Golang) to log transactions as described above. You must deploy the contract to the test network, which includes the newly added peer in Org1.

#### **Detailed Steps:**

**Part 1: Add a New Peer to Org1**

1. Navigate to the `test-network` directory in your `lxp-supplychain-hlf` folder.
2. Add a new peer (`peer1.org1.example.com`) to Org1. Make sure that:
   * The new peer has the correct Docker configuration.
   * You update the `docker-compose` file to include the new peer.
   * The new peer is fully operational and can be joined to the existing channel `mychannel`.
3. After adding the new peer, join `peer1.org1.example.com` to the `mychannel` channel.
4. Verify that the peer is successfully added by querying the channel to check if `peer1.org1.example.com` is listed as a member.

**Part 2: Write a Smart Contract**

1. Implement a smart contract called `TransactionLog` in Go (Golang) with the following functionalities:
   * **LogTransaction(transactionID, sender, receiver, amount):** This function logs a transaction with a unique ID, sender, receiver, and amount.
   * **QueryTransaction(transactionID):** This function retrieves a specific transaction using its ID.
   * **QueryAllTransactions():** This function returns all transactions logged in the system.
2. Deploy the smart contract on the `mychannel` that includes both `peer0.org1.example.com` and `peer1.org1.example.com`. You must install and instantiate the smart contract on both peers of Org1.

#### Notes:

The following steps are already performed for you as prerequisites:

* The Hyperledger Fabric test network can be started using the command `./network.sh up createChannel` with a default channel `mychannel`.
* The peers for `Org1` and `Org2` are joined to the channel.
* The chaincode environment setup is pre-configured in the `chaincode` directory.
*   Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn transactionlog -ccp ../chaincode -ccl go
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
source ./scripts/setOrgPeerContext.sh 1  # Org1 context

peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n transactionlog \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["LogTransaction","tx12345","Alice","Bob","100"]}'
```

```sh
source ./scripts/setOrgPeerContext.sh 1  # Org1 context

peer chaincode query -C mychannel -n transactionlog \
    -c '{"Args":["QueryTransaction","tx12345"]}'
```

You need to write docker files and  smart contract code and deploy it.
