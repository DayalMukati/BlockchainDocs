# Scoring Script

```
#!/usr/bin/env bash

# Start with Partial Credit: 0
echo "Partial Credit: 0"

# Navigate to the test network directory
cd ./test-network

# Set Org1 as the peer
source ./scripts/setOrgPeerContext.sh 1

# Ensure jq is installed for JSON parsing
if ! command -v jq &> /dev/null; then
    echo "Error: jq is not installed. Install it using: sudo apt install jq"
    exit 1
fi

SCORE=0

# Step 1: Check if `crowdfundchannel` exists
echo "Checking if channel 'crowdfundchannel' is created..."
CHANNEL_LIST=$(peer channel list 2>&1)
if echo "$CHANNEL_LIST" | grep -q "crowdfundchannel"; then
    SCORE=$((SCORE + 20))
    echo "✅ Channel 'crowdfundchannel' exists - +20 points"
else
    echo "❌ Channel 'crowdfundchannel' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if both Org1 and Org2 peers joined `crowdfundchannel`
echo "Checking if Org1 and Org2 peers have joined 'crowdfundchannel'..."
PEER1_INFO=$(peer channel getinfo -c crowdfundchannel 2>&1)
source ./scripts/setOrgPeerContext.sh 2
PEER2_INFO=$(peer channel getinfo -c crowdfundchannel 2>&1)

if echo "$PEER1_INFO" | grep -q "height"; then
    if echo "$PEER2_INFO" | grep -q "height"; then
        SCORE=$((SCORE + 20))
        echo "✅ Org1 and Org2 joined 'crowdfundchannel' - +20 points"
    else
        echo "❌ Org2 did not join 'crowdfundchannel'"
        echo "Partial Credit: $SCORE"
        exit 0
    fi
else
    echo "❌ Org1 did not join 'crowdfundchannel'"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if chaincode `crowdfundcc` is deployed
echo "Checking if chaincode 'crowdfundcc' is deployed..."
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_INFO" | grep -q "crowdfundcc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'crowdfundcc' is deployed - +20 points"
else
    echo "❌ Chaincode 'crowdfundcc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if a campaign was created
echo "Querying created campaign 'camp1'..."
CAMPAIGN_OUTPUT=$(peer chaincode query -C crowdfundchannel -n crowdfundcc -c '{"Args":["GetCampaign","camp1"]}' 2>&1)

if echo "$CAMPAIGN_OUTPUT" | jq -e '.campaignID == "camp1"' > /dev/null; then
    SCORE=$((SCORE + 20))
    echo "✅ Campaign 'camp1' created successfully - +20 points"
else
    echo "❌ Campaign 'camp1' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Check if funds were contributed
echo "Checking if funds were contributed..."
CURRENT_AMOUNT=$(echo "$CAMPAIGN_OUTPUT" | jq -r '.currentAmount')

if [[ "$CURRENT_AMOUNT" -ge 5000 ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ Funds contributed successfully - +10 points"
else
    echo "❌ Funds not contributed or incorrect amount"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 6: Check if funds were withdrawn by the creator
echo "Checking if funds were withdrawn..."
WITHDRAWN_STATUS=$(echo "$CAMPAIGN_OUTPUT" | jq -r '.withdrawn')

if [[ "$WITHDRAWN_STATUS" == "true" ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ Funds successfully withdrawn by the project creator - +10 points"
else
    echo "❌ Funds were not withdrawn"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Partial Credit Score
echo "Partial Credit: $SCORE"

```

