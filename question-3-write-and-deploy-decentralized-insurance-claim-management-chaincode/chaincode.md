# Chaincode

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract defines the Insurance Claim contract
type SmartContract struct {
	contractapi.Contract
}

// User represents a policyholder or insurer
type User struct {
	UserID   string `json:"userID"`
	Name     string `json:"name"`
	UserType string `json:"userType"` // "Policyholder" or "Insurer"
}

// Claim represents an insurance claim
type Claim struct {
	ClaimID     string `json:"claimID"`
	UserID      string `json:"userID"`
	ClaimAmount int    `json:"claimAmount"`
	ClaimReason string `json:"claimReason"`
	Status      string `json:"status"` // "Pending", "Approved", "Rejected"
	RejectionReason string `json:"rejectionReason,omitempty"`
}

// RegisterUser registers a new user (Policyholder or Insurer)
func (s *SmartContract) RegisterUser(ctx contractapi.TransactionContextInterface, userID string, name string, userType string) error {
	existingUser, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return fmt.Errorf("failed to read user from world state: %v", err)
	}
	if existingUser != nil {
		return fmt.Errorf("user already exists")
	}

	user := User{
		UserID:   userID,
		Name:     name,
		UserType: userType,
	}

	userJSON, err := json.Marshal(user)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(userID, userJSON)
}

// FileClaim allows a policyholder to file an insurance claim
func (s *SmartContract) FileClaim(ctx contractapi.TransactionContextInterface, claimID string, userID string, claimAmount int, claimReason string) error {
	existingClaim, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return fmt.Errorf("failed to read claim from world state: %v", err)
	}
	if existingClaim != nil {
		return fmt.Errorf("claim already exists")
	}

	userJSON, err := ctx.GetStub().GetState(userID)
	if err != nil {
		return fmt.Errorf("failed to read user: %v", err)
	}
	if userJSON == nil {
		return fmt.Errorf("user does not exist")
	}

	var user User
	err = json.Unmarshal(userJSON, &user)
	if err != nil {
		return err
	}

	if user.UserType != "Policyholder" {
		return fmt.Errorf("only policyholders can file claims")
	}

	claim := Claim{
		ClaimID:     claimID,
		UserID:      userID,
		ClaimAmount: claimAmount,
		ClaimReason: claimReason,
		Status:      "Pending",
	}

	claimJSON, err := json.Marshal(claim)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(claimID, claimJSON)
}

// ApproveClaim allows an insurer to approve a claim
func (s *SmartContract) ApproveClaim(ctx contractapi.TransactionContextInterface, claimID string) error {
	claimJSON, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return fmt.Errorf("failed to read claim: %v", err)
	}
	if claimJSON == nil {
		return fmt.Errorf("claim does not exist")
	}

	var claim Claim
	err = json.Unmarshal(claimJSON, &claim)
	if err != nil {
		return err
	}

	if claim.Status != "Pending" {
		return fmt.Errorf("claim is already processed")
	}

	claim.Status = "Approved"

	updatedClaimJSON, err := json.Marshal(claim)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(claimID, updatedClaimJSON)
}

// RejectClaim allows an insurer to reject a claim with a reason
func (s *SmartContract) RejectClaim(ctx contractapi.TransactionContextInterface, claimID string, rejectionReason string) error {
	claimJSON, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return fmt.Errorf("failed to read claim: %v", err)
	}
	if claimJSON == nil {
		return fmt.Errorf("claim does not exist")
	}

	var claim Claim
	err = json.Unmarshal(claimJSON, &claim)
	if err != nil {
		return err
	}

	if claim.Status != "Pending" {
		return fmt.Errorf("claim is already processed")
	}

	claim.Status = "Rejected"
	claim.RejectionReason = rejectionReason

	updatedClaimJSON, err := json.Marshal(claim)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(claimID, updatedClaimJSON)
}

// GetClaimStatus retrieves the status of a claim
func (s *SmartContract) GetClaimStatus(ctx contractapi.TransactionContextInterface, claimID string) (*Claim, error) {
	claimJSON, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return nil, fmt.Errorf("failed to read claim: %v", err)
	}
	if claimJSON == nil {
		return nil, fmt.Errorf("claim does not exist")
	}

	var claim Claim
	err = json.Unmarshal(claimJSON, &claim)
	if err != nil {
		return nil, err
	}

	return &claim, nil
}

// GetUserClaims retrieves all claims filed by a specific user
func (s *SmartContract) GetUserClaims(ctx contractapi.TransactionContextInterface, userID string) ([]Claim, error) {
	queryString := fmt.Sprintf(`{"selector":{"userID":"%s"}}`, userID)
	resultsIterator, err := ctx.GetStub().GetQueryResult(queryString)
	if err != nil {
		return nil, fmt.Errorf("failed to get user claims: %v", err)
	}
	defer resultsIterator.Close()

	var claims []Claim
	for resultsIterator.HasNext() {
		queryResponse, err := resultsIterator.Next()
		if err != nil {
			return nil, err
		}

		var claim Claim
		err = json.Unmarshal(queryResponse.Value, &claim)
		if err != nil {
			return nil, err
		}
		claims = append(claims, claim)
	}

	return claims, nil
}

// Main function to start the chaincode
func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating insurance claim chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting insurance claim chaincode: %s", err)
	}
}

```

