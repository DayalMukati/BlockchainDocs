# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract provides functions for simple energy trading
type SmartContract struct {
	contractapi.Contract
}

// User represents an energy producer or consumer
type User struct {
	UserID     string `json:"userID"`
	UserType   string `json:"userType"` // "Producer" or "Consumer"
	Balance    int    `json:"balance"`
	EnergyUnits int   `json:"energyUnits"`
}

// RegisterUser registers a new user with an initial token balance and energy units
func (s *SmartContract) RegisterUser(ctx contractapi.TransactionContextInterface, userID string, userType string, balance int) error {
	exists, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return fmt.Errorf("failed to check user existence: %v", err)
	}
	if exists != nil {
		return fmt.Errorf("user already exists")
	}

	user := User{
		UserID:     userID,
		UserType:   userType,
		Balance:    balance,
		EnergyUnits: 0, // Start with zero energy
	}

	userJSON, err := json.Marshal(user)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(userID, userJSON)
}

// ProduceEnergy allows a producer to generate energy
func (s *SmartContract) ProduceEnergy(ctx contractapi.TransactionContextInterface, producerID string, amount int) error {
	producerJSON, err := ctx.GetStub().GetState(producerID)
	if err != nil {
		return fmt.Errorf("failed to read producer state: %v", err)
	}
	if producerJSON == nil {
		return fmt.Errorf("producer does not exist")
	}

	var producer User
	err = json.Unmarshal(producerJSON, &producer)
	if err != nil {
		return err
	}

	if producer.UserType != "Producer" {
		return fmt.Errorf("only producers can generate energy")
	}

	producer.EnergyUnits += amount

	updatedProducerJSON, err := json.Marshal(producer)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(producerID, updatedProducerJSON)
}

// TransferEnergy allows a consumer to buy energy from a producer using tokens
func (s *SmartContract) TransferEnergy(ctx contractapi.TransactionContextInterface, producerID string, consumerID string, energyAmount int, tokenPrice int) error {
	producerJSON, err := ctx.GetStub().GetState(producerID)
	if err != nil {
		return fmt.Errorf("failed to read producer state: %v", err)
	}
	if producerJSON == nil {
		return fmt.Errorf("producer does not exist")
	}

	consumerJSON, err := ctx.GetStub().GetState(consumerID)
	if err != nil {
		return fmt.Errorf("failed to read consumer state: %v", err)
	}
	if consumerJSON == nil {
		return fmt.Errorf("consumer does not exist")
	}

	var producer User
	err = json.Unmarshal(producerJSON, &producer)
	if err != nil {
		return err
	}

	var consumer User
	err = json.Unmarshal(consumerJSON, &consumer)
	if err != nil {
		return err
	}

	if producer.EnergyUnits < energyAmount {
		return fmt.Errorf("producer does not have enough energy")
	}

	if consumer.Balance < tokenPrice {
		return fmt.Errorf("consumer does not have enough tokens")
	}

	// Transfer energy and tokens
	producer.EnergyUnits -= energyAmount
	consumer.EnergyUnits += energyAmount
	consumer.Balance -= tokenPrice
	producer.Balance += tokenPrice

	// Save updated states
	producerJSON, err = json.Marshal(producer)
	if err != nil {
		return err
	}

	consumerJSON, err = json.Marshal(consumer)
	if err != nil {
		return err
	}

	err = ctx.GetStub().PutState(producerID, producerJSON)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(consumerID, consumerJSON)
}

// GetUserDetails retrieves full user details
func (s *SmartContract) GetUserDetails(ctx contractapi.TransactionContextInterface, userID string) (*User, error) {
	userJSON, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return nil, fmt.Errorf("failed to read user state: %v", err)
	}
	if userJSON == nil {
		return nil, fmt.Errorf("user does not exist")
	}

	var user User
	err = json.Unmarshal(userJSON, &user)
	if err != nil {
		return nil, err
	}

	return &user, nil
}

// Main function to start the chaincode
func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating energy trade chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting energy trade chaincode: %s", err)
	}
}

```

