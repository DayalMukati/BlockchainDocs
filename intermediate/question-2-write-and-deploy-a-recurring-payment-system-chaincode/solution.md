# Solution

## Solution:

**Smart Contract:** recurringpaymen&#x74;**`.go`**

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type RecurringPaymentContract struct {
	contractapi.Contract
}

type Subscription struct {
	SubscriptionID   string `json:"subscriptionID"`
	PayerID          string `json:"payerID"`
	PayeeID          string `json:"payeeID"`
	Amount           float64 `json:"amount"`
	TotalInstallments int    `json:"totalInstallments"`
	PaymentsMade     int    `json:"paymentsMade"`
	Frequency        string `json:"frequency"`
	Completed        bool   `json:"completed"`
}

// CreateSubscription can only be invoked by Org1 users
func (rpc *RecurringPaymentContract) CreateSubscription(ctx contractapi.TransactionContextInterface, subscriptionID string, payerID string, payeeID string, amount float64, frequency string, totalInstallments int) error {
	// Verify that the transaction is submitted by Org1
	clientOrgID, err := ctx.GetClientIdentity().GetMSPID()
	if err != nil {
		return fmt.Errorf("failed to get client org: %s", err.Error())
	}
	if clientOrgID != "Org1MSP" {
		return fmt.Errorf("only Org1 can create a subscription")
	}

	subscription := Subscription{
		SubscriptionID:   subscriptionID,
		PayerID:          payerID,
		PayeeID:          payeeID,
		Amount:           amount,
		TotalInstallments: totalInstallments,
		PaymentsMade:     0,
		Frequency:        frequency,
		Completed:        false,
	}

	subscriptionAsBytes, err := json.Marshal(subscription)
	if err != nil {
		return fmt.Errorf("failed to marshal subscription: %s", err.Error())
	}

	return ctx.GetStub().PutState(subscriptionID, subscriptionAsBytes)
}

// MakePayment can only be invoked by Org1 users
func (rpc *RecurringPaymentContract) MakePayment(ctx contractapi.TransactionContextInterface, subscriptionID string) error {
	// Verify that the transaction is submitted by Org1
	clientOrgID, err := ctx.GetClientIdentity().GetMSPID()
	if err != nil {
		return fmt.Errorf("failed to get client org: %s", err.Error())
	}
	if clientOrgID != "Org1MSP" {
		return fmt.Errorf("only Org1 can make payments")
	}

	subscriptionAsBytes, err := ctx.GetStub().GetState(subscriptionID)
	if err != nil {
		return fmt.Errorf("failed to get subscription: %s", err.Error())
	}

	if subscriptionAsBytes == nil {
		return fmt.Errorf("subscription %s does not exist", subscriptionID)
	}

	subscription := new(Subscription)
	err = json.Unmarshal(subscriptionAsBytes, subscription)
	if err != nil {
		return fmt.Errorf("failed to unmarshal subscription: %s", err.Error())
	}

	if subscription.Completed {
		return fmt.Errorf("subscription %s is already completed", subscriptionID)
	}

	// Check if there are remaining installments
	if subscription.PaymentsMade >= subscription.TotalInstallments {
		return fmt.Errorf("no remaining installments for subscription %s", subscriptionID)
	}

	// Increment the number of payments made
	subscription.PaymentsMade++

	// If all payments are made, mark the subscription as completed
	if subscription.PaymentsMade >= subscription.TotalInstallments {
		subscription.Completed = true
	}

	subscriptionAsBytes, err = json.Marshal(subscription)
	if err != nil {
		return fmt.Errorf("failed to marshal subscription: %s", err.Error())
	}

	return ctx.GetStub().PutState(subscriptionID, subscriptionAsBytes)
}

// ConfirmPayment can only be invoked by Org2 users
func (rpc *RecurringPaymentContract) ConfirmPayment(ctx contractapi.TransactionContextInterface, subscriptionID string) error {
	// Verify that the transaction is submitted by Org2
	clientOrgID, err := ctx.GetClientIdentity().GetMSPID()
	if err != nil {
		return fmt.Errorf("failed to get client org: %s", err.Error())
	}
	if clientOrgID != "Org2MSP" {
		return fmt.Errorf("only Org2 can confirm payments")
	}

	subscriptionAsBytes, err := ctx.GetStub().GetState(subscriptionID)
	if err != nil {
		return fmt.Errorf("failed to get subscription: %s", err.Error())
	}

	if subscriptionAsBytes == nil {
		return fmt.Errorf("subscription %s does not exist", subscriptionID)
	}

	// If the payments made are confirmed, just print confirmation
	subscription := new(Subscription)
	err = json.Unmarshal(subscriptionAsBytes, subscription)
	if err != nil {
		return fmt.Errorf("failed to unmarshal subscription: %s", err.Error())
	}

	fmt.Printf("Payment for subscription %s confirmed by payee.\n", subscriptionID)
	return nil
}

// QuerySubscriptionStatus can only be invoked by Org2 users
func (rpc *RecurringPaymentContract) QuerySubscriptionStatus(ctx contractapi.TransactionContextInterface, subscriptionID string) (*Subscription, error) {
	// Verify that the transaction is submitted by Org2
	clientOrgID, err := ctx.GetClientIdentity().GetMSPID()
	if err != nil {
		return nil, fmt.Errorf("failed to get client org: %s", err.Error())
	}
	if clientOrgID != "Org2MSP" {
		return nil, fmt.Errorf("only Org2 can query the subscription status")
	}

	subscriptionAsBytes, err := ctx.GetStub().GetState(subscriptionID)
	if err != nil {
		return nil, fmt.Errorf("failed to get subscription: %s", err.Error())
	}

	if subscriptionAsBytes == nil {
		return nil, fmt.Errorf("subscription %s does not exist", subscriptionID)
	}

	subscription := new(Subscription)
	err = json.Unmarshal(subscriptionAsBytes, subscription)
	if err != nil {
		return nil, fmt.Errorf("failed to unmarshal subscription: %s", err.Error())
	}

	return subscription, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(new(RecurringPaymentContract))
	if err != nil {
		fmt.Printf("Error creating recurring payment contract: %s", err.Error())
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting recurring payment contract: %s", err.Error())
	}
}

```

### Steps to Deploy and Test:

1.  Go Inside lxp-recurringpayment-hlf/test-network and Start the test network if it’s not running:

    <pre class="language-bash"><code class="lang-bash">cd lxp-recurringpayment-hlf/test-network
    <strong>./network.sh up createChannel
    </strong></code></pre>
2.  Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn recurringpaymentorgs -ccp ../chaincode -ccl go
    ```
3. Create a subscription (Org1 user):

```bash
source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

```
source ./scripts/setOrgPeerContext.sh 1  # Org1 context

peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n recurringpaymentorgs \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateSubscription","sub1","payer1","payee1","1000","monthly","6"]}'
```

```
Output: INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

4. Make a payment (Org1 user):

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n recurringpaymentorgs \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["MakePayment","sub1"]}'

```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

5. Confirm the payment (Org2 user):

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
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

6. Query the subscription status (Org2 user):

```bash
peer chaincode query -C mychannel -n recurringpaymentorgs -c '{"Args":["QuerySubscriptionStatus","sub1"]}'
```

```
{"subscriptionID":"sub1","payerID":"payer1","payeeID":"payee1","amount":1000,"totalInstallments":6,"paymentsMade":1,"frequency":"monthly","completed":false}
```

