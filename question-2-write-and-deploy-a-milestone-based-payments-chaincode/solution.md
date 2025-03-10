# Solution

## Solution:

**Smart Contract: `milestone.go`**

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type MilestonePaymentContract struct {
	contractapi.Contract
}

type Milestone struct {
	MilestoneID string  `json:"milestoneID"`
	Amount      float64 `json:"amount"`
	Status      string  `json:"status"`
}

type Order struct {
	OrderID    string              `json:"orderID"`
	Milestones map[string]Milestone `json:"milestones"`
}

func (s *MilestonePaymentContract) CreateMilestonePayment(ctx contractapi.TransactionContextInterface, orderID string, milestoneID string, amount float64) error {
	orderAsBytes, err := ctx.GetStub().GetState(orderID)
	var order Order
	if err != nil || orderAsBytes == nil {
		order = Order{
			OrderID:    orderID,
			Milestones: make(map[string]Milestone),
		}
	} else {
		err = json.Unmarshal(orderAsBytes, &order)
		if err != nil {
			return fmt.Errorf("Failed to unmarshal order: %s", err.Error())
		}
	}

	order.Milestones[milestoneID] = Milestone{
		MilestoneID: milestoneID,
		Amount:      amount,
		Status:      "pending",
	}

	orderAsBytes, err = json.Marshal(order)
	if err != nil {
		return fmt.Errorf("Failed to marshal order: %s", err.Error())
	}

	return ctx.GetStub().PutState(orderID, orderAsBytes)
}

func (s *MilestonePaymentContract) UpdateMilestoneStatus(ctx contractapi.TransactionContextInterface, orderID string, milestoneID string, status string) error {
	orderAsBytes, err := ctx.GetStub().GetState(orderID)
	if err != nil || orderAsBytes == nil {
		return fmt.Errorf("Order %s does not exist", orderID)
	}

	var order Order
	err = json.Unmarshal(orderAsBytes, &order)
	if err != nil {
		return fmt.Errorf("Failed to unmarshal order: %s", err.Error())
	}

	milestone, exists := order.Milestones[milestoneID]
	if !exists {
		return fmt.Errorf("Milestone %s does not exist", milestoneID)
	}

	milestone.Status = status
	order.Milestones[milestoneID] = milestone

	orderAsBytes, err = json.Marshal(order)
	if err != nil {
		return fmt.Errorf("Failed to marshal order: %s", err.Error())
	}

	return ctx.GetStub().PutState(orderID, orderAsBytes)
}

func (s *MilestonePaymentContract) ReleaseMilestonePayment(ctx contractapi.TransactionContextInterface, orderID string, milestoneID string) error {
	orderAsBytes, err := ctx.GetStub().GetState(orderID)
	if err != nil || orderAsBytes == nil {
		return fmt.Errorf("Order %s does not exist", orderID)
	}

	var order Order
	err = json.Unmarshal(orderAsBytes, &order)
	if err != nil {
		return fmt.Errorf("Failed to unmarshal order: %s", err.Error())
	}

	milestone, exists := order.Milestones[milestoneID]
	if !exists {
		return fmt.Errorf("Milestone %s does not exist", milestoneID)
	}

	if milestone.Status != "completed" {
		return fmt.Errorf("Milestone %s is not completed", milestoneID)
	}

	fmt.Printf("Releasing payment of %.2f for milestone %s of order %s\n", milestone.Amount, milestoneID, orderID)

	milestone.Status = "paid"
	order.Milestones[milestoneID] = milestone

	orderAsBytes, err = json.Marshal(order)
	if err != nil {
		return fmt.Errorf("Failed to marshal order: %s", err.Error())
	}

	return ctx.GetStub().PutState(orderID, orderAsBytes)
}

func (s *MilestonePaymentContract) QueryMilestonePayments(ctx contractapi.TransactionContextInterface, orderID string) (map[string]Milestone, error) {
	orderAsBytes, err := ctx.GetStub().GetState(orderID)
	if err != nil || orderAsBytes == nil {
		return nil, fmt.Errorf("Order %s does not exist", orderID)
	}

	var order Order
	err = json.Unmarshal(orderAsBytes, &order)
	if err != nil {
		return nil, fmt.Errorf("Failed to unmarshal order: %s", err.Error())
	}

	return order.Milestones, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(new(MilestonePaymentContract))
	if err != nil {
		fmt.Printf("Error creating milestone payment chaincode: %s", err.Error())
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting milestone payment chaincode: %s", err.Error())
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
    ./network.sh deployCC -c mychannel -ccn milestonepayment -ccp ../chaincode -ccl go
    ```
3. Create a milestone payment:

```bash
source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n milestonepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateMilestonePayment","order1","milestone1","1000"]}'
```

```
Output: INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

4. Update the milestone status:

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n milestonepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["UpdateMilestoneStatus","order1","milestone1","completed"]}'

```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

5. Release the milestone payment:

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n milestonepayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ReleaseMilestonePayment","order1","milestone1"]}'
```

5. Query the milestone payments:

```bash
peer chaincode query -C mychannel -n milestonepayment -c '{"Args":["QueryMilestonePayments","order1"]}'
```

```
{"milestone1":{"milestoneID":"milestone1","amount":1000,"status":"paid"}}
```
