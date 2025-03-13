# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"
	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Cross-Border Payment Settlement
type SmartContract struct {
	contractapi.Contract
}

// Payment represents a cross-border payment transaction
type Payment struct {
	TxID      string `json:"txID"`
	Sender    string `json:"sender"`
	Receiver  string `json:"receiver"`
	Amount    int    `json:"amount"`
	Status    string `json:"status"` // "Initiated", "Approved", "Settled"
}

// InitiatePayment creates a new cross-border payment transaction
func (s *SmartContract) InitiatePayment(ctx contractapi.TransactionContextInterface, txID string, sender string, receiver string, amount int) error {
	existingTx, err := ctx.GetStub().GetState(txID)
	if err != nil {
		return fmt.Errorf("failed to check transaction existence: %v", err)
	}
	if existingTx != nil {
		return fmt.Errorf("transaction already exists")
	}

	payment := Payment{
		TxID:     txID,
		Sender:   sender,
		Receiver: receiver,
		Amount:   amount,
		Status:   "Initiated",
	}

	paymentJSON, err := json.Marshal(payment)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(txID, paymentJSON)
}

// ApprovePayment allows the receiving bank to approve the payment
func (s *SmartContract) ApprovePayment(ctx contractapi.TransactionContextInterface, txID string) error {
	paymentJSON, err := ctx.GetStub().GetState(txID)
	if err != nil {
		return fmt.Errorf("failed to read transaction: %v", err)
	}
	if paymentJSON == nil {
		return fmt.Errorf("transaction %s does not exist", txID)
	}

	var payment Payment
	err = json.Unmarshal(paymentJSON, &payment)
	if err != nil {
		return err
	}

	payment.Status = "Approved"

	updatedPaymentJSON, err := json.Marshal(payment)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(txID, updatedPaymentJSON)
}

// SettlePayment finalizes the transaction upon approval
func (s *SmartContract) SettlePayment(ctx contractapi.TransactionContextInterface, txID string) error {
	paymentJSON, err := ctx.GetStub().GetState(txID)
	if err != nil {
		return fmt.Errorf("failed to read transaction: %v", err)
	}
	if paymentJSON == nil {
		return fmt.Errorf("transaction %s does not exist", txID)
	}

	var payment Payment
	err = json.Unmarshal(paymentJSON, &payment)
	if err != nil {
		return err
	}

	if payment.Status != "Approved" {
		return fmt.Errorf("payment cannot be settled unless approved")
	}

	payment.Status = "Settled"

	updatedPaymentJSON, err := json.Marshal(payment)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(txID, updatedPaymentJSON)
}

// GetPaymentDetails retrieves transaction details
func (s *SmartContract) GetPaymentDetails(ctx contractapi.TransactionContextInterface, txID string) (*Payment, error) {
	paymentJSON, err := ctx.GetStub().GetState(txID)
	if err != nil {
		return nil, fmt.Errorf("failed to read transaction: %v", err)
	}
	if paymentJSON == nil {
		return nil, fmt.Errorf("transaction %s does not exist", txID)
	}

	var payment Payment
	err = json.Unmarshal(paymentJSON, &payment)
	if err != nil {
		return nil, err
	}

	return &payment, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating payment chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting payment chaincode: %s", err)
	}
}

```

