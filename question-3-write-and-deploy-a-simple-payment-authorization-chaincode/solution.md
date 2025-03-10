# Solution

## Solution:

**Smart Contract: `simplepayment.go`**

```go

package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type PaymentContract struct {
	contractapi.Contract
}

type Payer struct {
	PayerID string  `json:"payerID"`
	Limit   float64 `json:"limit"`
	Status  string  `json:"status"` // Only keeps track of the last status (PaymentAuthorized, PaymentFailed)
}

// SetPaymentLimit sets the payment limit for a payer
func (p *PaymentContract) SetPaymentLimit(ctx contractapi.TransactionContextInterface, payerID string, limit float64) error {
	payer := Payer{
		PayerID: payerID,
		Limit:   limit,
		Status:  "LimitSet",
	}

	payerAsBytes, err := json.Marshal(payer)
	if err != nil {
		return fmt.Errorf("failed to marshal payer: %s", err.Error())
	}

	return ctx.GetStub().PutState(payerID, payerAsBytes)
}

// AuthorizePayment authorizes a payment for a payer if it doesn't exceed the limit
func (p *PaymentContract) AuthorizePayment(ctx contractapi.TransactionContextInterface, payerID string, paymentAmount float64) error {
	payerAsBytes, err := ctx.GetStub().GetState(payerID)
	if err != nil {
		return fmt.Errorf("failed to get payer: %s", err.Error())
	}

	if payerAsBytes == nil {
		return fmt.Errorf("payer %s does not exist", payerID)
	}

	payer := new(Payer)
	err = json.Unmarshal(payerAsBytes, payer)
	if err != nil {
		return fmt.Errorf("failed to unmarshal payer: %s", err.Error())
	}

	// Check if payment exceeds the limit
	if paymentAmount > payer.Limit {
		payer.Status = "PaymentFailed"

		// Update status to failed and save back to the ledger
		payerAsBytes, _ = json.Marshal(payer)
		return ctx.GetStub().PutState(payerID, payerAsBytes)
	}

	// If payment is within the limit, set status to authorized
	payer.Status = "PaymentAuthorized"
	payerAsBytes, err = json.Marshal(payer)
	if err != nil {
		return fmt.Errorf("failed to marshal payer: %s", err.Error())
	}

	return ctx.GetStub().PutState(payerID, payerAsBytes)
}

// QueryPaymentStatus returns the last status of the payer
func (p *PaymentContract) QueryPaymentStatus(ctx contractapi.TransactionContextInterface, payerID string) (string, error) {
	payerAsBytes, err := ctx.GetStub().GetState(payerID)
	if err != nil {
		return "", fmt.Errorf("failed to get payer: %s", err.Error())
	}

	if payerAsBytes == nil {
		return "", fmt.Errorf("payer %s does not exist", payerID)
	}

	payer := new(Payer)
	err = json.Unmarshal(payerAsBytes, payer)
	if err != nil {
		return "", fmt.Errorf("failed to unmarshal payer: %s", err.Error())
	}

	// Return the current status of the payer
	return payer.Status, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(new(PaymentContract))
	if err != nil {
		fmt.Printf("Error creating payment contract chaincode: %s", err.Error())
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting payment contract chaincode: %s", err.Error())
	}
}
```

#### Steps to Deploy and Test:

1.  Start the test network if it’s not running:

    ```bash
    ./network.sh up createChannel
    ```
2.  Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn simplepayment -ccp ../chaincode -ccl go
    ```
3. Set a payment limit for `payer1`:

```bash
source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n simplepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["SetPaymentLimit","payer1","5000"]}'
```

```
Output: INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

4. Authorize a payment for `payer1`:

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n simplepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["AuthorizePayment","payer1","3000"]}'
```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

5. Try to authorize a payment exceeding the limit.

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n simplepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["AuthorizePayment","payer1","6000"]}'
```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

6. Query the payment limit for `payer1`:

```bash
peer chaincode query -C mychannel -n simplepayment -c '{"Args":["QueryPaymentLimit","payer1"]}'
```

```
PaymentFailed
```
