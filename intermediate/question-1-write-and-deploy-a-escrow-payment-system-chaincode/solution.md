# Solution

## Solution:

**Smart Contract: `escrowpayment.go`**

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type EscrowContract struct {
	contractapi.Contract
}

type Escrow struct {
	EscrowID   string  `json:"escrowID"`
	PayerID    string  `json:"payerID"`
	PayeeID    string  `json:"payeeID"`
	Amount     float64 `json:"amount"`
	PayerApproved bool `json:"payerApproved"`
	PayeeConfirmed bool `json:"payeeConfirmed"`
	Released   bool    `json:"released"`
}

func (e *EscrowContract) CreateEscrow(ctx contractapi.TransactionContextInterface, escrowID string, payerID string, payeeID string, amount float64) error {
	escrow := Escrow{
		EscrowID:   escrowID,
		PayerID:    payerID,
		PayeeID:    payeeID,
		Amount:     amount,
		PayerApproved: false,
		PayeeConfirmed: false,
		Released:   false,
	}

	escrowAsBytes, err := json.Marshal(escrow)
	if err != nil {
		return fmt.Errorf("failed to marshal escrow: %s", err.Error())
	}

	return ctx.GetStub().PutState(escrowID, escrowAsBytes)
}

func (e *EscrowContract) ApproveEscrow(ctx contractapi.TransactionContextInterface, escrowID string, approverID string) error {
	escrowAsBytes, err := ctx.GetStub().GetState(escrowID)
	if err != nil {
		return fmt.Errorf("failed to get escrow: %s", err.Error())
	}

	if escrowAsBytes == nil {
		return fmt.Errorf("escrow %s does not exist", escrowID)
	}

	escrow := new(Escrow)
	err = json.Unmarshal(escrowAsBytes, escrow)
	if err != nil {
		return fmt.Errorf("failed to unmarshal escrow: %s", err.Error())
	}

	// Approve the escrow for the payer
	if approverID == escrow.PayerID {
		escrow.PayerApproved = true
	} else if approverID == escrow.PayeeID {
		escrow.PayeeConfirmed = true
	} else {
		return fmt.Errorf("approver is neither payer nor payee")
	}

	escrowAsBytes, err = json.Marshal(escrow)
	if err != nil {
		return fmt.Errorf("failed to marshal escrow: %s", err.Error())
	}

	return ctx.GetStub().PutState(escrowID, escrowAsBytes)
}

func (e *EscrowContract) ReleaseEscrow(ctx contractapi.TransactionContextInterface, escrowID string) error {
	escrowAsBytes, err := ctx.GetStub().GetState(escrowID)
	if err != nil {
		return fmt.Errorf("failed to get escrow: %s", err.Error())
	}

	if escrowAsBytes == nil {
		return fmt.Errorf("escrow %s does not exist", escrowID)
	}

	escrow := new(Escrow)
	err = json.Unmarshal(escrowAsBytes, escrow)
	if err != nil {
		return fmt.Errorf("failed to unmarshal escrow: %s", err.Error())
	}

	// Ensure both payer and payee have approved
	if !escrow.PayerApproved || !escrow.PayeeConfirmed {
		return fmt.Errorf("both parties have not approved the escrow")
	}

	escrow.Released = true

	escrowAsBytes, err = json.Marshal(escrow)
	if err != nil {
		return fmt.Errorf("failed to marshal escrow: %s", err.Error())
	}

	return ctx.GetStub().PutState(escrowID, escrowAsBytes)
}

func (e *EscrowContract) QueryEscrowStatus(ctx contractapi.TransactionContextInterface, escrowID string) (*Escrow, error) {
	escrowAsBytes, err := ctx.GetStub().GetState(escrowID)
	if err != nil {
		return nil, fmt.Errorf("failed to get escrow: %s", err.Error())
	}

	if escrowAsBytes == nil {
		return nil, fmt.Errorf("escrow %s does not exist", escrowID)
	}

	escrow := new(Escrow)
	err = json.Unmarshal(escrowAsBytes, escrow)
	if err != nil {
		return nil, fmt.Errorf("failed to unmarshal escrow: %s", err.Error())
	}

	return escrow, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(new(EscrowContract))
	if err != nil {
		fmt.Printf("Error creating escrow contract: %s", err.Error())
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting escrow contract: %s", err.Error())
	}
}

```

```go
```

### Steps to Deploy and Test:

1.  Go Inside lxp-suuplychain-hlf/test-network and Start the test network if it’s not running:

    ```bash
    cd xlp-supplychain-hlf/test-network
    ./network.sh up createChannel
    ```
2.  Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn escrowpayment -ccp ../chaincode -ccl go
    ```
3. Create an escrow account:

```bash
source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["CreateEscrow","escrow1","payer1","payee1","1000"]}'
```

```
Output: INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

4. Approve the escrow as the payer1:

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApproveEscrow","escrow1","payer1"]}'
```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

5. Confirm the escrow as the payee (`payee1`):

```
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ApproveEscrow","escrow1","payee1"]}'
```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

6. Release the escrow funds:

```bash
peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n escrowpayment \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["ReleaseEscrow","escrow1"]}'
```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

7. Query the escrow status:

```
peer chaincode query -C mychannel -n escrowpayment -c '{"Args":["QueryEscrowStatus","escrow1"]}'
```

```
{"escrowID":"escrow1","payerID":"payer1","payeeID":"payee1","amount":1000,"payerApproved":true,"payeeConfirmed":true,"released":true}
```
