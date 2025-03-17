# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"
	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Trade Finance
type SmartContract struct {
	contractapi.Contract
}

// LetterOfCredit represents a trade finance transaction
type LetterOfCredit struct {
	LoCID       string `json:"locID"`
	Importer    string `json:"importer"`
	Exporter    string `json:"exporter"`
	Amount      int    `json:"amount"`
	ApprovedBy  string `json:"approvedBy"`
	Approved    bool   `json:"approved"`
	ShipmentDone bool  `json:"shipmentDone"`
	Settled     bool   `json:"settled"`
}

// RequestLoC allows an importer to request a Letter of Credit
func (s *SmartContract) RequestLoC(ctx contractapi.TransactionContextInterface, locID string, importer string, exporter string, amount int) error {
	existingLoC, err := ctx.GetStub().GetState(locID)
	if err != nil {
		return fmt.Errorf("failed to check LoC existence: %v", err)
	}
	if existingLoC != nil {
		return fmt.Errorf("LoC already exists")
	}

	loc := LetterOfCredit{
		LoCID:       locID,
		Importer:    importer,
		Exporter:    exporter,
		Amount:      amount,
		ApprovedBy:  "",
		Approved:    false,
		ShipmentDone: false,
		Settled:     false,
	}

	locJSON, err := json.Marshal(loc)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(locID, locJSON)
}

// ApproveLoC allows a bank to approve the LoC
func (s *SmartContract) ApproveLoC(ctx contractapi.TransactionContextInterface, locID string, bank string) error {
	locJSON, err := ctx.GetStub().GetState(locID)
	if err != nil {
		return fmt.Errorf("failed to read LoC: %v", err)
	}
	if locJSON == nil {
		return fmt.Errorf("LoC %s does not exist", locID)
	}

	var loc LetterOfCredit
	err = json.Unmarshal(locJSON, &loc)
	if err != nil {
		return err
	}

	loc.ApprovedBy = bank
	loc.Approved = true

	updatedLoCJSON, err := json.Marshal(loc)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(locID, updatedLoCJSON)
}

// CompleteShipment allows an exporter to mark shipment as completed
func (s *SmartContract) CompleteShipment(ctx contractapi.TransactionContextInterface, locID string) error {
	locJSON, err := ctx.GetStub().GetState(locID)
	if err != nil {
		return fmt.Errorf("failed to read LoC: %v", err)
	}
	if locJSON == nil {
		return fmt.Errorf("LoC %s does not exist", locID)
	}

	var loc LetterOfCredit
	err = json.Unmarshal(locJSON, &loc)
	if err != nil {
		return err
	}

	loc.ShipmentDone = true

	updatedLoCJSON, err := json.Marshal(loc)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(locID, updatedLoCJSON)
}

// SettleLoC marks an LoC as settled once shipment is verified
func (s *SmartContract) SettleLoC(ctx contractapi.TransactionContextInterface, locID string) error {
	locJSON, err := ctx.GetStub().GetState(locID)
	if err != nil {
		return fmt.Errorf("failed to read LoC: %v", err)
	}
	if locJSON == nil {
		return fmt.Errorf("LoC %s does not exist", locID)
	}

	var loc LetterOfCredit
	err = json.Unmarshal(locJSON, &loc)
	if err != nil {
		return err
	}

	loc.Settled = true

	updatedLoCJSON, err := json.Marshal(loc)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(locID, updatedLoCJSON)
}

// GetLoC retrieves LoC details
func (s *SmartContract) GetLoC(ctx contractapi.TransactionContextInterface, locID string) (*LetterOfCredit, error) {
	locJSON, err := ctx.GetStub().GetState(locID)
	if err != nil {
		return nil, fmt.Errorf("failed to read LoC: %v", err)
	}
	if locJSON == nil {
		return nil, fmt.Errorf("LoC %s does not exist", locID)
	}

	var loc LetterOfCredit
	err = json.Unmarshal(locJSON, &loc)
	if err != nil {
		return nil, err
	}

	return &loc, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating trade finance chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting trade finance chaincode: %s", err)
	}
}

```

