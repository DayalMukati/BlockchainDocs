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

# Step 1: Check if `tradechannel` exists
echo "Checking if channel 'tradechannel' is created..."
CHANNEL_LIST=$(peer channel list 2>&1)
if echo "$CHANNEL_LIST" | grep -q "tradechannel"; then
    SCORE=$((SCORE + 20))
    echo "✅ Channel 'tradechannel' exists - +20 points"
else
    echo "❌ Channel 'tradechannel' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if both Org1 and Org2 peers joined `tradechannel`
echo "Checking if Org1 and Org2 peers have joined 'tradechannel'..."
PEER1_INFO=$(peer channel getinfo -c tradechannel 2>&1)
source ./scripts/setOrgPeerContext.sh 2
PEER2_INFO=$(peer channel getinfo -c tradechannel 2>&1)

if echo "$PEER1_INFO" | grep -q "height"; then
    if echo "$PEER2_INFO" | grep -q "height"; then
        SCORE=$((SCORE + 20))
        echo "✅ Org1 and Org2 joined 'tradechannel' - +20 points"
    else
        echo "❌ Org2 did not join 'tradechannel'"
        echo "Partial Credit: $SCORE"
        exit 0
    fi
else
    echo "❌ Org1 did not join 'tradechannel'"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if chaincode `tradecc` is deployed
echo "Checking if chaincode 'tradecc' is deployed..."
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_INFO" | grep -q "tradecc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'tradecc' is deployed - +20 points"
else
    echo "❌ Chaincode 'tradecc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if a Letter of Credit (LoC) was requested
echo "Querying requested LoC 'loc1'..."
LOC_OUTPUT=$(peer chaincode query -C tradechannel -n tradecc -c '{"Args":["GetLoC","loc1"]}' 2>&1)

if echo "$LOC_OUTPUT" | jq -e '.locID == "loc1"' > /dev/null; then
    SCORE=$((SCORE + 10))
    echo "✅ Letter of Credit 'loc1' requested successfully - +10 points"
else
    echo "❌ LoC 'loc1' request not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Check if LoC was approved
echo "Checking if LoC was approved by the bank..."
LOC_APPROVED=$(echo "$LOC_OUTPUT" | jq -r '.approved')

if [[ "$LOC_APPROVED" == "true" ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ LoC 'loc1' was approved by the bank - +10 points"
else
    echo "❌ LoC 'loc1' approval not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 6: Check if shipment was completed
echo "Checking if shipment was marked as completed..."
SHIPMENT_STATUS=$(echo "$LOC_OUTPUT" | jq -r '.shipmentDone')

if [[ "$SHIPMENT_STATUS" == "true" ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ Shipment completed successfully - +10 points"
else
    echo "❌ Shipment completion not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 7: Check if LoC was settled
echo "Checking if LoC was settled..."
LOC_SETTLED=$(echo "$LOC_OUTPUT" | jq -r '.settled')

if [[ "$LOC_SETTLED" == "true" ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ LoC successfully settled - +10 points"
else
    echo "❌ LoC settlement not completed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Partial Credit Score
echo "Partial Credit: $SCORE"

```

