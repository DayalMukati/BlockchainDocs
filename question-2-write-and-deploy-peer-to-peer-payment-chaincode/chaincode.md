# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Payments
type SmartContract struct {
	contractapi.Contract
}

// Account represents a bank account
type Account struct {
	AccountID string `json:"accountID"`
	Name      string `json:"name"`
	Balance   int    `json:"balance"`
}

// RegisterAccount creates a new account
func (s *SmartContract) RegisterAccount(ctx contractapi.TransactionContextInterface, accountID string, name string, balanceStr string) error {
	balance, err := strconv.Atoi(balanceStr)
	if err != nil {
		return fmt.Errorf("invalid balance value")
	}

	existingAccount, err := ctx.GetStub().GetState(accountID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if existingAccount != nil {
		return fmt.Errorf("account %s already exists", accountID)
	}

	account := Account{
		AccountID: accountID,
		Name:      name,
		Balance:   balance,
	}

	accountJSON, err := json.Marshal(account)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(accountID, accountJSON)
}

// TransferFunds securely transfers funds from one account to another
func (s *SmartContract) TransferFunds(ctx contractapi.TransactionContextInterface, fromAccount string, toAccount string, amountStr string) error {
	amount, err := strconv.Atoi(amountStr)
	if err != nil {
		return fmt.Errorf("invalid transfer amount")
	}

	fromAccJSON, err := ctx.GetStub().GetState(fromAccount)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if fromAccJSON == nil {
		return fmt.Errorf("source account %s does not exist", fromAccount)
	}

	toAccJSON, err := ctx.GetStub().GetState(toAccount)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if toAccJSON == nil {
		return fmt.Errorf("destination account %s does not exist", toAccount)
	}

	var fromAcc Account
	var toAcc Account
	err = json.Unmarshal(fromAccJSON, &fromAcc)
	if err != nil {
		return err
	}

	err = json.Unmarshal(toAccJSON, &toAcc)
	if err != nil {
		return err
	}

	if fromAcc.Balance < amount {
		return fmt.Errorf("insufficient funds in account %s", fromAccount)
	}

	fromAcc.Balance -= amount
	toAcc.Balance += amount

	updatedFromAccJSON, err := json.Marshal(fromAcc)
	if err != nil {
		return err
	}
	updatedToAccJSON, err := json.Marshal(toAcc)
	if err != nil {
		return err
	}

	err = ctx.GetStub().PutState(fromAccount, updatedFromAccJSON)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(toAccount, updatedToAccJSON)
}

// GetBalance retrieves the balance of an account
func (s *SmartContract) GetBalance(ctx contractapi.TransactionContextInterface, accountID string) (int, error) {
	accountJSON, err := ctx.GetStub().GetState(accountID)
	if err != nil {
		return 0, fmt.Errorf("failed to read account: %v", err)
	}
	if accountJSON == nil {
		return 0, fmt.Errorf("account %s does not exist", accountID)
	}

	var account Account
	err = json.Unmarshal(accountJSON, &account)
	if err != nil {
		return 0, err
	}

	return account.Balance, nil
}

// Main function to start the chaincode
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

