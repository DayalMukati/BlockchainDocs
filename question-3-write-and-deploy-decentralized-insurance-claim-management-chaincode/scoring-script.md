# Scoring Script

```bash
#!/usr/bin/env bash

# Start with Partial Credit: 0
echo "Partial Credit: 0"

# Initialize challenge evaluation
. /blackbox/blackbox docker-stdl check "challenge"

# Navigate to the test network directory
cd ./test-network

# Set Org1 as the peer
source ./scripts/setOrgPeerContext.sh 1

# Update PATH to include necessary binaries
export FABRIC_CFG_PATH=${PWD}/configtx

SCORE=0  # Initialize SCORE variable

# Step 1: Check if chaincode 'insurancecc' is installed
echo "Checking if chaincode 'insurancecc' is installed..."
CHAINCODE_CHECK=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_CHECK" | grep -q "insurancecc"; then
    SCORE=$((SCORE + 30))
    echo "Insurance claim chaincode 'insurancecc' is installed - +30 points"
else
    echo "Insurance claim chaincode 'insurancecc' is not installed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Query policyholder details (Fix: Use GetPolicyholderDetails instead of GetClaimDetails)
POLICYHOLDER_DETAILS=$(peer chaincode query -C mychannel -n insurancecc -c '{"Args":["GetPolicyholderDetails","P123"]}' 2>&1)

# Check if policyholder exists
if echo "$POLICYHOLDER_DETAILS" | grep -q "Error"; then
    echo "Error: Policyholder 'P123' does not exist."
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Verify that a claim was successfully filed
CLAIM_DETAILS=$(peer chaincode query -C mychannel -n insurancecc -c '{"Args":["GetClaimDetails","claim1"]}' 2>&1)

if echo "$CLAIM_DETAILS" | grep -q "Error"; then
    echo "Error: Claim 'claim1' does not exist."
    echo "Partial Credit: $SCORE"
    exit 0
else
    SCORE=$((SCORE + 20))
    echo "Claim 'claim1' successfully filed - +20 points"
fi

# Step 4: Check if claim was approved
if echo "$CLAIM_DETAILS" | grep -q '"status": "Approved"'; then
    SCORE=$((SCORE + 20))
    echo "Claim 'claim1' was approved - +20 points"
else
    echo "Error: Claim was not approved."
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Verify policyholder received payout
UPDATED_POLICYHOLDER_DETAILS=$(peer chaincode query -C mychannel -n insurancecc -c '{"Args":["GetPolicyholderDetails","P123"]}' 2>&1)

if echo "$UPDATED_POLICYHOLDER_DETAILS" | grep -q '"balance": 6000'; then
    SCORE=$((SCORE + 20))
    echo "Policyholder received claim payout - +20 points"
else
    echo "Error: Policyholder did not receive claim payout."
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Print Final Score
echo "Partial Credit: $SCORE"

```

