# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Invoice Management
type SmartContract struct {
	contractapi.Contract
}

// Invoice represents an invoice record
type Invoice struct {
	InvoiceID string `json:"invoiceID"`
	Buyer     string `json:"buyer"`
	Seller    string `json:"seller"`
	Amount    int    `json:"amount"`
	DueDate   string `json:"dueDate"`
	Status    string `json:"status"` // "Pending", "Approved", "Settled", "Disputed"
}

// CreateInvoice generates a new invoice
func (s *SmartContract) CreateInvoice(ctx contractapi.TransactionContextInterface, invoiceID string, buyer string, seller string, amount int, dueDate string) error {
	existingInvoice, err := ctx.GetStub().GetState(invoiceID)
	if err != nil {
		return fmt.Errorf("failed to check invoice existence: %v", err)
	}
	if existingInvoice != nil {
		return fmt.Errorf("invoice already exists")
	}

	invoice := Invoice{
		InvoiceID: invoiceID,
		Buyer:     buyer,
		Seller:    seller,
		Amount:    amount,
		DueDate:   dueDate,
		Status:    "Pending",
	}

	invoiceJSON, err := json.Marshal(invoice)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(invoiceID, invoiceJSON)
}

// ApproveInvoice allows the buyer to approve the invoice
func (s *SmartContract) ApproveInvoice(ctx contractapi.TransactionContextInterface, invoiceID string) error {
	invoiceJSON, err := ctx.GetStub().GetState(invoiceID)
	if err != nil {
		return fmt.Errorf("failed to read invoice: %v", err)
	}
	if invoiceJSON == nil {
		return fmt.Errorf("invoice %s does not exist", invoiceID)
	}

	var invoice Invoice
	err = json.Unmarshal(invoiceJSON, &invoice)
	if err != nil {
		return err
	}

	invoice.Status = "Approved"

	updatedInvoiceJSON, err := json.Marshal(invoice)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(invoiceID, updatedInvoiceJSON)
}

// SettleInvoice marks an invoice as paid
func (s *SmartContract) SettleInvoice(ctx contractapi.TransactionContextInterface, invoiceID string) error {
	invoiceJSON, err := ctx.GetStub().GetState(invoiceID)
	if err != nil {
		return fmt.Errorf("failed to read invoice: %v", err)
	}
	if invoiceJSON == nil {
		return fmt.Errorf("invoice %s does not exist", invoiceID)
	}

	var invoice Invoice
	err = json.Unmarshal(invoiceJSON, &invoice)
	if err != nil {
		return err
	}

	if invoice.Status != "Approved" {
		return fmt.Errorf("invoice cannot be settled unless approved")
	}

	invoice.Status = "Settled"

	updatedInvoiceJSON, err := json.Marshal(invoice)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(invoiceID, updatedInvoiceJSON)
}

// GetInvoiceDetails retrieves invoice details
func (s *SmartContract) GetInvoiceDetails(ctx contractapi.TransactionContextInterface, invoiceID string) (*Invoice, error) {
	invoiceJSON, err := ctx.GetStub().GetState(invoiceID)
	if err != nil {
		return nil, fmt.Errorf("failed to read invoice: %v", err)
	}
	if invoiceJSON == nil {
		return nil, fmt.Errorf("invoice %s does not exist", invoiceID)
	}

	var invoice Invoice
	err = json.Unmarshal(invoiceJSON, &invoice)
	if err != nil {
		return nil, err
	}

	return &invoice, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating invoice management chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting invoice management chaincode: %s", err)
	}
}

```

