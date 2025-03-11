# Chaincode

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Student Records
type SmartContract struct {
	contractapi.Contract
}

// Student represents a student record
type Student struct {
	StudentID string `json:"studentID"`
	Name      string `json:"name"`
	Age       int    `json:"age"`
	Course    string `json:"course"`
}

// RegisterStudent adds a new student record
func (s *SmartContract) RegisterStudent(ctx contractapi.TransactionContextInterface, studentID string, name string, age int, course string) error {
	existingStudent, err := ctx.GetStub().GetState(studentID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}
	if existingStudent != nil {
		return fmt.Errorf("student %s already exists", studentID)
	}

	student := Student{
		StudentID: studentID,
		Name:      name,
		Age:       age,
		Course:    course,
	}

	studentJSON, err := json.Marshal(student)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(studentID, studentJSON)
}

// UpdateCourse updates the course of an existing student
func (s *SmartContract) UpdateCourse(ctx contractapi.TransactionContextInterface, studentID string, newCourse string) error {
	studentJSON, err := ctx.GetStub().GetState(studentID)
	if err != nil {
		return fmt.Errorf("failed to read student record: %v", err)
	}
	if studentJSON == nil {
		return fmt.Errorf("student %s does not exist", studentID)
	}

	var student Student
	err = json.Unmarshal(studentJSON, &student)
	if err != nil {
		return err
	}

	student.Course = newCourse

	updatedStudentJSON, err := json.Marshal(student)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(studentID, updatedStudentJSON)
}

// GetStudent retrieves the details of a student
func (s *SmartContract) GetStudent(ctx contractapi.TransactionContextInterface, studentID string) (*Student, error) {
	studentJSON, err := ctx.GetStub().GetState(studentID)
	if err != nil {
		return nil, fmt.Errorf("failed to read student record: %v", err)
	}
	if studentJSON == nil {
		return nil, fmt.Errorf("student %s does not exist", studentID)
	}

	var student Student
	err = json.Unmarshal(studentJSON, &student)
	if err != nil {
		return nil, err
	}

	return &student, nil
}

// Main function to start the chaincode
func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating student record chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting student record chaincode: %s", err)
	}
}

```

