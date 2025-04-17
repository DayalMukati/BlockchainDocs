# Chaincode

```go
package main

import (
	"encoding/json"
	"fmt"
	"strconv"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract provides functions for managing insurance claims
type SmartContract struct {
	contractapi.Contract
}

// Policyholder represents a user with an insurance policy
type Policyholder struct {
	PolicyID string `json:"policyID"`
	Name     string `json:"name"`
	Balance  int    `json:"balance"`
}

// Claim represents an insurance claim filed by a policyholder
type Claim struct {
	ClaimID  string `json:"claimID"`
	PolicyID string `json:"policyID"`
	Amount   int    `json:"amount"`
	Reason   string `json:"reason"`
	Status   string `json:"status"` // "Pending", "Approved", "Rejected"
}

// RegisterPolicyholder creates a new policyholder record
func (s *SmartContract) RegisterPolicyholder(ctx contractapi.TransactionContextInterface, policyID string, name string, balanceStr string) error {
	_, err := ctx.GetStub().GetState(policyID)
	if err != nil {
		return fmt.Errorf("failed to read from world state: %v", err)
	}

	balance, err := strconv.Atoi(balanceStr)
	if err != nil {
		return fmt.Errorf("invalid balance amount: %v", err)
	}

	holder := Policyholder{
		PolicyID: policyID,
		Name:     name,
		Balance:  balance,
	}

	data, err := json.Marshal(holder)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(policyID, data)
}

// FileClaim allows a policyholder to file an insurance claim
func (s *SmartContract) FileClaim(ctx contractapi.TransactionContextInterface, claimID, policyID, amountStr, reason string) error {
	_, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return fmt.Errorf("failed to read claim state: %v", err)
	}

	amount, err := strconv.Atoi(amountStr)
	if err != nil {
		return fmt.Errorf("invalid amount: %v", err)
	}

	claim := Claim{
		ClaimID:  claimID,
		PolicyID: policyID,
		Amount:   amount,
		Reason:   reason,
		Status:   "Pending",
	}

	claimJSON, err := json.Marshal(claim)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(claimID, claimJSON)
}

// ReviewClaim allows the insurer to approve or reject a claim
func (s *SmartContract) ReviewClaim(ctx contractapi.TransactionContextInterface, claimID string, approve string) error {
	claimJSON, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return fmt.Errorf("failed to get claim: %v", err)
	}
	if claimJSON == nil {
		return fmt.Errorf("claim %s does not exist", claimID)
	}

	var claim Claim
	err = json.Unmarshal(claimJSON, &claim)
	if err != nil {
		return err
	}

	// Update status
	if approve == "true" {
		claim.Status = "Approved"

		// Credit amount to policyholder
		holderJSON, err := ctx.GetStub().GetState(claim.PolicyID)
		if err != nil {
			return fmt.Errorf("failed to get policyholder: %v", err)
		}
		if holderJSON == nil {
			return fmt.Errorf("policyholder %s does not exist", claim.PolicyID)
		}

		var holder Policyholder
		err = json.Unmarshal(holderJSON, &holder)
		if err != nil {
			return err
		}

		holder.Balance += claim.Amount

		holderUpdated, err := json.Marshal(holder)
		if err != nil {
			return err
		}

		err = ctx.GetStub().PutState(claim.PolicyID, holderUpdated)
		if err != nil {
			return err
		}

	} else {
		claim.Status = "Rejected"
	}

	claimUpdated, err := json.Marshal(claim)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(claimID, claimUpdated)
}

// GetClaimDetails retrieves details of a claim
func (s *SmartContract) GetClaimDetails(ctx contractapi.TransactionContextInterface, claimID string) (*Claim, error) {
	claimJSON, err := ctx.GetStub().GetState(claimID)
	if err != nil {
		return nil, fmt.Errorf("failed to read claim: %v", err)
	}
	if claimJSON == nil {
		return nil, fmt.Errorf("claim %s does not exist", claimID)
	}

	var claim Claim
	err = json.Unmarshal(claimJSON, &claim)
	if err != nil {
		return nil, err
	}

	return &claim, nil
}

// GetPolicyholderDetails retrieves details of a policyholder
func (s *SmartContract) GetPolicyholderDetails(ctx contractapi.TransactionContextInterface, policyID string) (*Policyholder, error) {
	holderJSON, err := ctx.GetStub().GetState(policyID)
	if err != nil {
		return nil, fmt.Errorf("failed to read policyholder: %v", err)
	}
	if holderJSON == nil {
		return nil, fmt.Errorf("policyholder %s does not exist", policyID)
	}

	var holder Policyholder
	err = json.Unmarshal(holderJSON, &holder)
	if err != nil {
		return nil, err
	}

	return &holder, nil
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

