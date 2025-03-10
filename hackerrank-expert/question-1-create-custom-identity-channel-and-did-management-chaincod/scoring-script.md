# Scoring Script

```

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

# Step 1: Check if `identitychannel` exists
echo "Checking if channel 'identitychannel' is created..."
CHANNEL_LIST=$(peer channel list 2>&1)

if echo "$CHANNEL_LIST" | grep -q "identitychannel"; then
    SCORE=$((SCORE + 20))
    echo "✅ Channel 'identitychannel' exists - +20 points"
else
    echo "❌ Channel 'identitychannel' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if both Org1 and Org2 peers joined `identitychannel`
echo "Checking if Org1 and Org2 peers have joined 'identitychannel'..."
PEER1_INFO=$(peer channel getinfo -c identitychannel 2>&1)
source ./scripts/setOrgPeerContext.sh 2
PEER2_INFO=$(peer channel getinfo -c identitychannel 2>&1)

if echo "$PEER1_INFO" | grep -q "height"; then
    if echo "$PEER2_INFO" | grep -q "height"; then
        SCORE=$((SCORE + 20))
        echo "✅ Org1 and Org2 joined 'identitychannel' - +20 points"
    else
        echo "❌ Org2 did not join 'identitychannel'"
        echo "Partial Credit: $SCORE"
        exit 0
    fi
else
    echo "❌ Org1 did not join 'identitychannel'"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if chaincode `identitycc` is deployed
echo "Checking if chaincode 'identitycc' is deployed..."
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_INFO" | grep -q "identitycc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'identitycc' is deployed - +20 points"
else
    echo "❌ Chaincode 'identitycc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if DID was created
echo "Querying for DID 'did:user1'..."
DID_OUTPUT=$(peer chaincode query -C identitychannel -n identitycc -c '{"Args":["GetDID","did:user1"]}' 2>&1)

if echo "$DID_OUTPUT" | grep -q '"did": "did:user1"'; then
    SCORE=$((SCORE + 10))
    echo "✅ DID 'did:user1' exists - +10 points"
else
    echo "❌ DID 'did:user1' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Check if DID credentials were updated
echo "Checking if DID credentials were updated..."
DID_CREDENTIALS=$(echo "$DID_OUTPUT" | grep -o '"credentials": "[^"]*"' | awk -F': ' '{print $2}' | tr -d '"')

if [[ "$DID_CREDENTIALS" == "Passport + Driving License Verified" ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ Credentials updated correctly - +10 points"
else
    echo "❌ Credentials not updated correctly. Found: $DID_CREDENTIALS"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 6: Check if DID was verified
echo "Checking if DID is verified..."
DID_VERIFIED=$(echo "$DID_OUTPUT" | grep -o '"verified": [^,}]*' | awk -F': ' '{print $2}')

if [[ "$DID_VERIFIED" == "true" ]]; then
    SCORE=$((SCORE + 10))
    echo "✅ DID is verified - +20 points"
else
    echo "❌ DID is not verified"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Partial Credit Score
echo "Partial Credit: $SCORE"
```

