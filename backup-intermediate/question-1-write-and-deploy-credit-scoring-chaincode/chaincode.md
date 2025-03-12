# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Credit Scoring & Loan Risk Management
type SmartContract struct {
	contractapi.Contract
}

// User represents a borrower
type User struct {
	UserID       string `json:"userID"`
	Name         string `json:"name"`
	CreditScore  int    `json:"creditScore"`
}

// Loan represents a loan request
type Loan struct {
	LoanID      string `json:"loanID"`
	UserID      string `json:"userID"`
	Amount      int    `json:"amount"`
	Balance     int    `json:"balance"`
	Status      string `json:"status"` // "Pending", "Approved", "Repaid"
}

// RegisterUser registers a user with an initial credit score
func (s *SmartContract) RegisterUser(ctx contractapi.TransactionContextInterface, userID string, name string, creditScoreStr string) error {
	creditScore, err := strconv.Atoi(creditScoreStr)
	if err != nil {
		return fmt.Errorf("invalid credit score")
	}

	user := User{
		UserID:      userID,
		Name:        name,
		CreditScore: creditScore,
	}

	userJSON, err := json.Marshal(user)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(userID, userJSON)
}

// UpdateCreditScore updates the credit score of a user
func (s *SmartContract) UpdateCreditScore(ctx contractapi.TransactionContextInterface, userID string, newScoreStr string) error {
	newScore, err := strconv.Atoi(newScoreStr)
	if err != nil {
		return fmt.Errorf("invalid credit score")
	}

	userJSON, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return fmt.Errorf("failed to read user: %v", err)
	}
	if userJSON == nil {
		return fmt.Errorf("user %s does not exist", userID)
	}

	var user User
	err = json.Unmarshal(userJSON, &user)
	if err != nil {
		return err
	}

	user.CreditScore = newScore

	updatedUserJSON, err := json.Marshal(user)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(userID, updatedUserJSON)
}

// IssueLoan issues a loan if the user meets the credit score requirement
func (s *SmartContract) IssueLoan(ctx contractapi.TransactionContextInterface, loanID string, userID string, amountStr string) error {
	amount, err := strconv.Atoi(amountStr)
	if err != nil {
		return fmt.Errorf("invalid loan amount")
	}

	userJSON, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return fmt.Errorf("failed to read user: %v", err)
	}
	if userJSON == nil {
		return fmt.Errorf("user %s does not exist", userID)
	}

	var user User
	err = json.Unmarshal(userJSON, &user)
	if err != nil {
		return err
	}

	if user.CreditScore < 700 {
		return fmt.Errorf("loan denied due to low credit score")
	}

	loan := Loan{
		LoanID:  loanID,
		UserID:  userID,
		Amount:  amount,
		Balance: amount,
		Status:  "Approved",
	}

	loanJSON, err := json.Marshal(loan)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(loanID, loanJSON)
}

// GetUserCreditHistory retrieves a user's credit history
func (s *SmartContract) GetUserCreditHistory(ctx contractapi.TransactionContextInterface, userID string) (*User, error) {
	userJSON, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return nil, fmt.Errorf("failed to read user: %v", err)
	}
	if userJSON == nil {
		return nil, fmt.Errorf("user %s does not exist", userID)
	}

	var user User
	err = json.Unmarshal(userJSON, &user)
	if err != nil {
		return nil, err
	}

	return &user, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating credit score chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting credit score chaincode: %s", err)
	}
}

```

