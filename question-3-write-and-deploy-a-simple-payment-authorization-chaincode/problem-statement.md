# Problem Statement

#### Problem Statement:

You are tasked with writing a simple smart contract for payment authorization on a Hyperledger Fabric network. The goal is to authorize payments only if the requested amount does not exceed the payer's allowed payment limit.

You are required to:

1. **Write a smart contract** that supports the following functionalities:
   * **SetPaymentLimit(payerID, limit)**: Set a maximum payment limit for a payer.
   * **AuthorizePayment(payerID, paymentAmount)**: Authorize a payment for a payer, ensuring that it does not exceed the payment limit. The payment will be processed and the status will either be `"PaymentAuthorized"` or `"PaymentFailed"`, depending on whether the payment exceeds the limit.
   * **QueryPaymentStatus(payerID)**: Retrieve the current status for a payer, indicating whether the last payment was authorized or failed.
2. **Deploy this smart contract** on a test network that is already set up with a default channel.

#### Testing the Smart Contract:

Use the provided pre-configured test network, which includes the following:

* Two peers, one for Org1 and one for Org2.
* A default channel named `mychannel`.
* A pre-initialized ledger with no data.

#### Test the smart contract by performing the following actions:

1. **Set the payment limit** for `payer1` at 5000.
2. **Authorize a payment** of 3000 for `payer1` (this should be successful).
3. **Attempt to authorize a payment** of 6000 for `payer1`. This will not throw an error but will update the status to `"PaymentFailed"`.
4. **Query the payment status** for `payer1` to verify the results of the payment attempts.

#### Requirements:

* The smart contract should be written in **Go (Golang)**.
* The test network should be used to test the contract, and test results should be displayed using `peer chaincode invoke` and `peer chaincode query`.

#### Notes:

The following steps are already performed for you as prerequisites:

* The Hyperledger Fabric test network is started using the command `./network.sh up createChannel` with a default channel `mychannel`.
* The peers for Org1 and Org2 are joined to the channel.
* The chaincode environment setup is pre-configured in the chaincode directory.

#### Deploy the Chaincode:

1.  To deploy the chaincode, use the following command:

    ```bash
    bashCopy code./network.sh deployCC -c mychannel -ccn simplepayment -ccp ../chaincode -ccl go
    ```
2.  A script is provided in the `scripts` folder to set the environment variables for Org1 and Org2 peers. You can use the following command to set the context for Org1:

    ```bash
    bashCopy codesource ./scripts/setOrgPeerContext.sh 1
    ```

#### Support Commands:

Use the following commands to test the contract:

1.  **Set the payment limit for `payer1`:**

    ```bash
    bashCopy codepeer chaincode invoke -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        --tls --cafile $ORDERER_CA -C mychannel -n simplepayment \
        --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
        --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
        -c '{"Args":["SetPaymentLimit","payer1","5000"]}'
    ```
2.  **Authorize a payment of 3000 for `payer1`:**

    ```bash
    bashCopy codepeer chaincode invoke -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        --tls --cafile $ORDERER_CA -C mychannel -n simplepayment \
        --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
        --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
        -c '{"Args":["AuthorizePayment","payer1","3000"]}'
    ```
3.  **Attempt to authorize a payment of 6000 for `payer1` (will update the status to `"PaymentFailed"`):**

    ```bash
    bashCopy codepeer chaincode invoke -o localhost:7050 \
        --ordererTLSHostnameOverride orderer.example.com \
        --tls --cafile $ORDERER_CA -C mychannel -n simplepayment \
        --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
        --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
        -c '{"Args":["AuthorizePayment","payer1","6000"]}'
    ```
4.  **Query the payment status for `payer1`:**

    ```bash
    bashCopy codepeer chaincode query -C mychannel -n simplepayment -c '{"Args":["QueryPaymentStatus","payer1"]}'
    ```

#### Expected Results:

1. **SetPaymentLimit**: The limit for `payer1` should be set to 5000.
2. **AuthorizePayment (3000)**: The payment of 3000 should be successful and the status should be `"PaymentAuthorized"`.
3. **AuthorizePayment (6000)**: The payment of 6000 should update the status to `"PaymentFailed"` due to exceeding the limit.
4. **QueryPaymentStatus**: The query should return the status of the last action for `payer1`, either `"PaymentAuthorized"` or `"PaymentFailed"`.

You only need to write the smart contract code, deploy it, and perform the test steps mentioned.
