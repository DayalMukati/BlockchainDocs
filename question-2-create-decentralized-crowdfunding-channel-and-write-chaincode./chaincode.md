# Chaincode

```
package main

import (
	"encoding/json"
	"fmt"
	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract for Crowdfunding
type SmartContract struct {
	contractapi.Contract
}

// Campaign represents a crowdfunding campaign
type Campaign struct {
	CampaignID  string `json:"campaignID"`
	Creator     string `json:"creator"`
	GoalAmount  int    `json:"goalAmount"`
	CurrentAmount int  `json:"currentAmount"`
	Withdrawn   bool   `json:"withdrawn"`
}

// CreateCampaign launches a new crowdfunding campaign
func (s *SmartContract) CreateCampaign(ctx contractapi.TransactionContextInterface, campaignID string, creator string, goalAmount int) error {
	existingCampaign, err := ctx.GetStub().GetState(campaignID)
	if err != nil {
		return fmt.Errorf("failed to check campaign existence: %v", err)
	}
	if existingCampaign != nil {
		return fmt.Errorf("campaign already exists")
	}

	campaign := Campaign{
		CampaignID:  campaignID,
		Creator:     creator,
		GoalAmount:  goalAmount,
		CurrentAmount: 0,
		Withdrawn:   false,
	}

	campaignJSON, err := json.Marshal(campaign)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(campaignID, campaignJSON)
}

// ContributeFunds allows backers to contribute funds to a campaign
func (s *SmartContract) ContributeFunds(ctx contractapi.TransactionContextInterface, campaignID string, amount int) error {
	campaignJSON, err := ctx.GetStub().GetState(campaignID)
	if err != nil {
		return fmt.Errorf("failed to read campaign: %v", err)
	}
	if campaignJSON == nil {
		return fmt.Errorf("campaign %s does not exist", campaignID)
	}

	var campaign Campaign
	err = json.Unmarshal(campaignJSON, &campaign)
	if err != nil {
		return err
	}

	campaign.CurrentAmount += amount

	updatedCampaignJSON, err := json.Marshal(campaign)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(campaignID, updatedCampaignJSON)
}

// WithdrawFunds allows the project creator to withdraw funds once the goal is met
func (s *SmartContract) WithdrawFunds(ctx contractapi.TransactionContextInterface, campaignID string) error {
	campaignJSON, err := ctx.GetStub().GetState(campaignID)
	if err != nil {
		return fmt.Errorf("failed to read campaign: %v", err)
	}
	if campaignJSON == nil {
		return fmt.Errorf("campaign %s does not exist", campaignID)
	}

	var campaign Campaign
	err = json.Unmarshal(campaignJSON, &campaign)
	if err != nil {
		return err
	}

	if campaign.CurrentAmount < campaign.GoalAmount {
		return fmt.Errorf("campaign goal not met, cannot withdraw funds")
	}

	campaign.Withdrawn = true

	updatedCampaignJSON, err := json.Marshal(campaign)
	if err != nil {
		return err
	}

	return ctx.GetStub().PutState(campaignID, updatedCampaignJSON)
}

// GetCampaign retrieves campaign details
func (s *SmartContract) GetCampaign(ctx contractapi.TransactionContextInterface, campaignID string) (*Campaign, error) {
	campaignJSON, err := ctx.GetStub().GetState(campaignID)
	if err != nil {
		return nil, fmt.Errorf("failed to read campaign: %v", err)
	}
	if campaignJSON == nil {
		return nil, fmt.Errorf("campaign %s does not exist", campaignID)
	}

	var campaign Campaign
	err = json.Unmarshal(campaignJSON, &campaign)
	if err != nil {
		return nil, err
	}

	return &campaign, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(&SmartContract{})
	if err != nil {
		fmt.Printf("Error creating crowdfunding chaincode: %s", err)
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting crowdfunding chaincode: %s", err)
	}
}

```

