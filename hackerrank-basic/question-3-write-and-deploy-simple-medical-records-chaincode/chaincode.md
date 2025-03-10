# Chaincode

```bash

package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract defines the Medical Records contract
type SmartContract struct {
	contractapi.Contract
}

// Patient represents a patient record in the ledger
type Patient struct {
	PatientID      string `json:"patientID"`
	Name           string `json:"name"`
	Age            int    `json:"age"`
	MedicalHistory string `json:"medicalHistory"`
}

// RegisterPatient creates a new patient record
func (s *SmartContract) RegisterPatient(ctx contractapi.TransactionContextInterface, patientID string, name string, age int, medicalHistory string) error {
	existingPatient, err := ctx.GetStub().GetState(patientID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if existingPatient != nil {
		return fmt.Errorf("patient already exists")
	}

	patient := Patient{
		PatientID:      patientID,
		Name:           name,
		Age:            age,
		MedicalHistory: medicalHistory,
	}

	patientJSON, err := json.Marshal(patient)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(patientID, patientJSON)
}

// UpdateMedicalHistory appends new history to a patient's record
func (s *SmartContract) UpdateMedicalHistory(ctx contractapi.TransactionContextInterface, patientID string, newHistory string) error {
	patientJSON, err := ctx.GetStub().GetState(patientID)
	if err != nil {
		return fmt.Errorf("failed to read patient: %v", err)
	}
	if patientJSON == nil {
		return fmt.Errorf("patient does not exist")
	}

	var patient Patient
	err = json.Unmarshal(patientJSON, &patient)
	if err != nil {
		return err
	}

	patient.MedicalHistory = patient.MedicalHistory + ", " + newHistory

	updatedPatientJSON, err := json.Marshal(patient)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(patientID, updatedPatientJSON)
}

// GetPatientDetails retrieves the details of a patient
func (s *SmartContract) GetPatientDetails(ctx contractapi.TransactionContextInterface, patientID string) (*Patient, error) {
	patientJSON, err := ctx.GetStub().GetState(patientID)
	if err != nil {
		return nil, fmt.Errorf("failed to read patient: %v", err)
	}
	if patientJSON == nil {
		return nil, fmt.Errorf("patient does not exist")
	}

	var patient Patient
	err = json.Unmarshal(patientJSON, &patient)
	if err != nil {
		return nil, err
	}

	return &patient, nil
}

// Main function to start the chaincode
func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating medical records chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting medical records chaincode: %s", err)
	}
}

```

