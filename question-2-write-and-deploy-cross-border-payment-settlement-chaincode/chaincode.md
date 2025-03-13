# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Payment Settlement
type SmartContract struct {
	contractapi.Contract
}

// Payment represents a cross-border payment transaction
type Payment struct {
	PaymentID   string `json:"paymentID"`
	SenderBank  string `json:"senderBank"`
	ReceiverBank string `json:"receiverBank"`
	Amount      int    `json:"amount"`
	Status      string `json:"status"` // "Initiated", "Approved", "Settled", "Refunded"
}

// InitiatePayment creates a new payment request
func (s *SmartContract) InitiatePayment(ctx contractapi.TransactionContextInterface, paymentID string, senderBank string, receiverBank string, amount int) error {
	existingPayment, err := ctx.GetStub().GetState(paymentID)
	if err != nil {
		return fmt.Errorf("failed to check payment existence: %v", err)
	}
	if existingPayment != nil {
		return fmt.Errorf("payment already exists")
	}

	payment := Payment{
		PaymentID:    paymentID,
		SenderBank:   senderBank,
		ReceiverBank: receiverBank,
		Amount:       amount,
		Status:       "Initiated",
	}

	paymentJSON, err := json.Marshal(payment)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(paymentID, paymentJSON)
}

// ApprovePayment approves a payment request
func (s *SmartContract) ApprovePayment(ctx contractapi.TransactionContextInterface, paymentID string) error {
	paymentJSON, err := ctx.GetStub().GetState(paymentID)
	if err != nil {
		return fmt.Errorf("failed to read payment: %v", err)
	}
	if paymentJSON == nil {
		return fmt.Errorf("payment %s does not exist", paymentID)
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

	return ctx.GetStub().PutState(paymentID, updatedPaymentJSON)
}

// SettlePayment completes the payment transaction
func (s *SmartContract) SettlePayment(ctx contractapi.TransactionContextInterface, paymentID string) error {
	paymentJSON, err := ctx.GetStub().GetState(paymentID)
	if err != nil {
		return fmt.Errorf("failed to read payment: %v", err)
	}
	if paymentJSON == nil {
		return fmt.Errorf("payment %s does not exist", paymentID)
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

	return ctx.GetStub().PutState(paymentID, updatedPaymentJSON)
}

// GetPaymentDetails retrieves payment details
func (s *SmartContract) GetPaymentDetails(ctx contractapi.TransactionContextInterface, paymentID string) (*Payment, error) {
	paymentJSON, err := ctx.GetStub().GetState(paymentID)
	if err != nil {
		return nil, fmt.Errorf("failed to read payment: %v", err)
	}
	if paymentJSON == nil {
		return nil, fmt.Errorf("payment %s does not exist", paymentID)
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
		fmt.Printf("Error creating payment settlement chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting payment settlement chaincode: %s", err)
	}
}

```

