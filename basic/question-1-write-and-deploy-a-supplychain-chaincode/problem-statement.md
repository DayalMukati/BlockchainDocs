# Problem Statement

## **Problem Statement:**

You are tasked with writing a simple supply chain smart contract for a Hyperledger Fabric network. The goal is to create a smart contract that manages the lifecycle of a product in the supply chain, including its creation, transfer between participants, and final delivery.

You are required to:

1. Write a smart contract that supports the following functionalities:
   * `CreateProduct(productID, productName, owner)`: Create a new product in the supply chain.
   * `TransferProduct(productID, newOwner)`: Transfer the ownership of the product to a new owner.
   * `GetProduct(productID)`: Retrieve the current state of the product.
2. Deploy this smart contract on a test network that is already set up with a default channel.
3. Use the provided pre-configured test network, which includes the following:
   * Two peers, one for `Org1` and one for `Org2`.
   * A default channel named `mychannel`.
   * A pre-initialized ledger with no data.
4. Test the smart contract by performing the following actions:
   * Create a product with ID `prod1`, name `CoffeeBeans`, and owner `SupplierA`.
   * Transfer the product ownership to `DistributorA`.
   * Retrieve and display the current state of the product.

**Requirements:**

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

Support Commands:&#x20;

<pre><code>source ./scripts/setOrgPeerContext.sh 1

#To Set the Path
export PATH=${PWD}/../bin:${PWD}:$PATH
<strong>export FABRIC_CFG_PATH=${PWD}/configtx
</strong><strong>
</strong><strong>peer chaincode invoke -o localhost:7050 \
</strong>    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n supplychain \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateProduct","prod1","CoffeeBeans","SupplierA"]}'

peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n supplychain \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["TransferProduct","prod1","DistributorA"]}'
    
peer chaincode query -C mychannel -n supplychain -c '{"Args":["GetProduct","prod1"]}'
</code></pre>



You only need to write the smart contract code and deploy it.
