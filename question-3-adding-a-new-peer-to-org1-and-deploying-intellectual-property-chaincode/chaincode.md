# Chaincode

```bash
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for IP Management
type SmartContract struct {
	contractapi.Contract
}

// IntellectualProperty represents an IP asset
type IntellectualProperty struct {
	IPID       string `json:"ipID"`
	Owner      string `json:"owner"`
	Title      string `json:"title"`
	Registered string `json:"registered"`
}

// RegisterIP creates a new Intellectual Property record
func (s *SmartContract) RegisterIP(ctx contractapi.TransactionContextInterface, ipID string, owner string, title string, registered string) error {
	existingIP, err := ctx.GetStub().GetState(ipID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if existingIP != nil {
		return fmt.Errorf("IP already exists")
	}

	ip := IntellectualProperty{
		IPID:       ipID,
		Owner:      owner,
		Title:      title,
		Registered: registered,
	}

	ipJSON, err := json.Marshal(ip)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(ipID, ipJSON)
}

// TransferIP transfers ownership of an IP
func (s *SmartContract) TransferIP(ctx contractapi.TransactionContextInterface, ipID string, newOwner string) error {
	ipJSON, err := ctx.GetStub().GetState(ipID)
	if err != nil {
		return fmt.Errorf("failed to read IP: %v", err)
	}
	if ipJSON == nil {
		return fmt.Errorf("IP does not exist")
	}

	var ip IntellectualProperty
	err = json.Unmarshal(ipJSON, &ip)
	if err != nil {
		return err
	}

	ip.Owner = newOwner

	updatedIPJSON, err := json.Marshal(ip)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(ipID, updatedIPJSON)
}

// VerifyOwnership retrieves the owner of an IP
func (s *SmartContract) VerifyOwnership(ctx contractapi.TransactionContextInterface, ipID string) (*IntellectualProperty, error) {
	ipJSON, err := ctx.GetStub().GetState(ipID)
	if err != nil {
		return nil, fmt.Errorf("failed to read IP: %v", err)
	}
	if ipJSON == nil {
		return nil, fmt.Errorf("IP does not exist")
	}

	var ip IntellectualProperty
	err = json.Unmarshal(ipJSON, &ip)
	if err != nil {
		return nil, err
	}

	return &ip, nil
}

// Main function to start the chaincode
func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating IP rights chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting IP rights chaincode: %s", err)
	}
}

```

