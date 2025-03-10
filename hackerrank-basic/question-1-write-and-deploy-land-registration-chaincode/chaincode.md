# Chaincode

```bash
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract defines the Land Registry contract
type SmartContract struct {
	contractapi.Contract
}

// Land represents a land record
type Land struct {
	LandID   string `json:"LandID"`
	Owner    string `json:"Owner"`
	Location string `json:"Location"`
}

// RegisterLand creates a new land record
func (s *SmartContract) RegisterLand(ctx contractapi.TransactionContextInterface, landID string, ownerName string, location string) error {
	existingLand, err := ctx.GetStub().GetState(landID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if existingLand != nil {
		return fmt.Errorf("land record already exists")
	}

	land := Land{
		LandID:   landID,
		Owner:    ownerName,
		Location: location,
	}

	landJSON, err := json.Marshal(land)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(landID, landJSON)
}

// TransferLandOwnership transfers ownership of a land record
func (s *SmartContract) TransferLandOwnership(ctx contractapi.TransactionContextInterface, landID string, newOwner string) error {
	landJSON, err := ctx.GetStub().GetState(landID)
	if err != nil {
		return fmt.Errorf("failed to read land record: %v", err)
	}
	if landJSON == nil {
		return fmt.Errorf("land record does not exist")
	}

	var land Land
	err = json.Unmarshal(landJSON, &land)
	if err != nil {
		return err
	}

	land.Owner = newOwner

	updatedLandJSON, err := json.Marshal(land)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(landID, updatedLandJSON)
}

// GetLandDetails retrieves the details of a land record
func (s *SmartContract) GetLandDetails(ctx contractapi.TransactionContextInterface, landID string) (*Land, error) {
	landJSON, err := ctx.GetStub().GetState(landID)
	if err != nil {
		return nil, fmt.Errorf("failed to read land record: %v", err)
	}
	if landJSON == nil {
		return nil, fmt.Errorf("land record does not exist")
	}

	var land Land
	err = json.Unmarshal(landJSON, &land)
	if err != nil {
		return nil, err
	}

	return &land, nil
}

// Main function to start the chaincode
func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating land registration chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting land registration chaincode: %s", err)
	}
}

```

