# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Loan Management
type SmartContract struct {
	contractapi.Contract
}

// Loan represents a loan request
type Loan struct {
	LoanID      string `json:"loanID"`
	BorrowerID  string `json:"borrowerID"`
	Amount      int    `json:"amount"`
	Balance     int    `json:"balance"`
	Status      string `json:"status"` // "Pending", "Approved", "Repaid"
}

// ApplyForLoan allows a borrower to apply for a loan
func (s *SmartContract) ApplyForLoan(ctx contractapi.TransactionContextInterface, loanID string, borrowerID string, amountStr string) error {
	amount, err := strconv.Atoi(amountStr)
	if err != nil {
		return fmt.Errorf("invalid loan amount")
	}

	existingLoan, err := ctx.GetStub().GetState(loanID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if existingLoan != nil {
		return fmt.Errorf("loan %s already exists", loanID)
	}

	loan := Loan{
		LoanID:     loanID,
		BorrowerID: borrowerID,
		Amount:     amount,
		Balance:    amount,
		Status:     "Pending",
	}

	loanJSON, err := json.Marshal(loan)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(loanID, loanJSON)
}

// ApproveLoan allows a lender to approve a loan
func (s *SmartContract) ApproveLoan(ctx contractapi.TransactionContextInterface, loanID string) error {
	loanJSON, err := ctx.GetStub().GetState(loanID)
	if err != nil {
		return fmt.Errorf("failed to read loan: %v", err)
	}
	if loanJSON == nil {
		return fmt.Errorf("loan %s does not exist", loanID)
	}

	var loan Loan
	err = json.Unmarshal(loanJSON, &loan)
	if err != nil {
		return err
	}

	loan.Status = "Approved"

	updatedLoanJSON, err := json.Marshal(loan)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(loanID, updatedLoanJSON)
}

// RepayLoan allows a borrower to repay part of the loan
func (s *SmartContract) RepayLoan(ctx contractapi.TransactionContextInterface, loanID string, amountStr string) error {
	amount, err := strconv.Atoi(amountStr)
	if err != nil {
		return fmt.Errorf("invalid repayment amount")
	}

	loanJSON, err := ctx.GetStub().GetState(loanID)
	if err != nil {
		return fmt.Errorf("failed to read loan: %v", err)
	}
	if loanJSON == nil {
		return fmt.Errorf("loan %s does not exist", loanID)
	}

	var loan Loan
	err = json.Unmarshal(loanJSON, &loan)
	if err != nil {
		return err
	}

	if loan.Balance < amount {
		return fmt.Errorf("repayment amount exceeds loan balance")
	}

	loan.Balance -= amount
	if loan.Balance == 0 {
		loan.Status = "Repaid"
	}

	updatedLoanJSON, err := json.Marshal(loan)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(loanID, updatedLoanJSON)
}

// GetLoan retrieves loan details
func (s *SmartContract) GetLoan(ctx contractapi.TransactionContextInterface, loanID string) (*Loan, error) {
	loanJSON, err := ctx.GetStub().GetState(loanID)
	if err != nil {
		return nil, fmt.Errorf("failed to read loan: %v", err)
	}
	if loanJSON == nil {
		return nil, fmt.Errorf("loan %s does not exist", loanID)
	}

	var loan Loan
	err = json.Unmarshal(loanJSON, &loan)
	if err != nil {
		return nil, err
	}

	return &loan, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating loan management chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting loan management chaincode: %s", err)
	}
}

```

