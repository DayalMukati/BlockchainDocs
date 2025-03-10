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

# Step 1: Check if `tokenizationchannel` exists
echo "Checking if channel 'tokenizationchannel' is created..."
CHANNEL_LIST=$(peer channel list 2>&1)

if echo "$CHANNEL_LIST" | grep -q "tokenizationchannel"; then
    SCORE=$((SCORE + 20))
    echo "✅ Channel 'tokenizationchannel' exists - +20 points"
else
    echo "❌ Channel 'tokenizationchannel' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if both Org1 and Org2 peers joined `tokenizationchannel`
echo "Checking if Org1 and Org2 peers have joined 'tokenizationchannel'..."
PEER1_INFO=$(peer channel getinfo -c tokenizationchannel 2>&1)
source ./scripts/setOrgPeerContext.sh 2
PEER2_INFO=$(peer channel getinfo -c tokenizationchannel 2>&1)

if echo "$PEER1_INFO" | grep -q "height"; then
    if echo "$PEER2_INFO" | grep -q "height"; then
        SCORE=$((SCORE + 20))
        echo "✅ Org1 and Org2 joined 'tokenizationchannel' - +20 points"
    else
        echo "❌ Org2 did not join 'tokenizationchannel'"
        echo "Partial Credit: $SCORE"
        exit 0
    fi
else
    echo "❌ Org1 did not join 'tokenizationchannel'"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if chaincode `tokencc` is deployed
echo "Checking if chaincode 'tokencc' is deployed..."
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_INFO" | grep -q "tokencc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'tokencc' is deployed - +20 points"
else
    echo "❌ Chaincode 'tokencc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if asset was tokenized
echo "Querying tokenized asset 'asset1'..."
TOKEN_OUTPUT=$(peer chaincode query -C tokenizationchannel -n tokencc -c '{"Args":["GetAsset","asset1"]}' 2>&1)

if echo "$TOKEN_OUTPUT" | grep -q '"assetID": "asset1"'; then
    SCORE=$((SCORE + 10))
    echo "✅ Asset 'asset1' was successfully tokenized - +10 points"
else
    echo "❌ Asset 'asset1' not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Check if tokens were successfully transferred
echo "Checking token transfer..."
if echo "$TOKEN_OUTPUT" | grep -q '"owner": "Org2"'; then
    SCORE=$((SCORE + 10))
    echo "✅ Tokens successfully transferred to Org2 - +10 points"
else
    echo "❌ Token transfer failed or incorrect owner"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 6: Check if tokens were burned
echo "Checking if tokens were burned..."
if echo "$TOKEN_OUTPUT" | grep -q '"status": "burned"'; then
    SCORE=$((SCORE + 10))
    echo "✅ Tokens were successfully burned - +10 points"
else
    echo "❌ Tokens were not burned"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Partial Credit Score
echo "Partial Credit: $SCORE"

```

