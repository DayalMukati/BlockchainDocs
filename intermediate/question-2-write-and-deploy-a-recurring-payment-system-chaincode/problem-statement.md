# Problem Statement

#### **Problem Statement:**

You are tasked with writing a smart contract for managing recurring payments on a Hyperledger Fabric network. The contract will handle subscription-based services where the payer (Org1) needs to make periodic payments to the payee (Org2) over a set duration. Only Org1 can make payments, and Org2 can confirm and query the status of subscriptions.

You are required to:

Write a smart contract that supports the following functionalities:

* **CreateSubscription(subscriptionID, payerID, payeeID, amount, frequency, totalInstallments):** This can only be done by Org1 users (payer).
* **MakePayment(subscriptionID):** This can only be done by Org1 users (payer).
* **ConfirmPayment(subscriptionID):** This can only be done by Org2 users (payee), confirming they’ve received the payment.
* **QuerySubscriptionStatus(subscriptionID):** This can only be done by Org2 users (payee), to check the current status of the subscription.

#### **Test the smart contract by performing the following actions:**

1. **Create a subscription** with ID `sub1`, where Org1's user (`payer1`) creates the subscription for Org2's user (`payee1`).
2. **Make a payment** for the subscription (`sub1`) as Org1's user (`payer1`).
3. **Confirm the payment** as Org2's user (`payee1`).
4. **Query the subscription status** as Org2's user (`payee1`).
5. **Make all payments** until the subscription is fully paid by Org1's user (`payer1`).
6. **Confirm each payment and query the status** after the final payment by Org2's user (`payee1`).

#### Notes:

The following steps are already performed for you as prerequisites:

* The Hyperledger Fabric test network can be started using the command `./network.sh up createChannel` with a default channel `mychannel`.
* The peers for `Org1` and `Org2` are joined to the channel.
* The chaincode environment setup is pre-configured in the `chaincode` directory.
*   Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn recurringpaymentorgs -ccp ../chaincode -ccl go
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
    --tls --cafile $ORDERER_CA -C mychannel -n recurringpaymentorgs \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateSubscription","sub1","payer1","payee1","1000","monthly","6"]}'
```

```sh
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n recurringpaymentorgs \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["MakePayment","sub1"]}'

```

```
source ./scripts/setOrgPeerContext.sh 2  # Org2 context

peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n recurringpaymentorgs \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ConfirmPayment","sub1"]}'

```

```
peer chaincode query -C mychannel -n recurringpaymentorgs -c '{"Args":["QuerySubscriptionStatus","sub1"]}'
```

You only need to write the smart contract code and deploy it.
