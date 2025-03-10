# Scoring Script

```bash
#!/usr/bin/env bash

# Start with Partial Credit: 0
echo "Partial Credit: 0"

# Initialize framework
. /blackbox/blackbox docker-stdl check "challenge"

# Navigate to the test network directory
cd ./test-network

# Set Org1 as the peer
source ./scripts/setOrgPeerContext.sh 1

# Ensure jq is installed
if ! command -v jq &> /dev/null; then
    echo "Error: jq is not installed. Install it using: sudo apt install jq"
    exit 1
fi

SCORE=0

# Step 1: Check if `peer1.org1.example.com` is running
echo "Checking if peer1.org1.example.com is running..."
PEER1_STATUS=$(docker ps | grep peer1.org1.example.com)
if [ -n "$PEER1_STATUS" ]; then
    SCORE=$((SCORE + 20))
    echo "✅ New peer 'peer1.org1.example.com' is running - +20 points"
else
    echo "❌ New peer 'peer1.org1.example.com' is not running"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if `peer1.org1.example.com` joined `ipchannel`
echo "Checking if peer1.org1.example.com joined 'ipchannel'..."
PEER_JOINED=$(peer channel list 2>&1)
if echo "$PEER_JOINED" | grep -q "ipchannel"; then
    SCORE=$((SCORE + 20))
    echo "✅ New peer joined 'ipchannel' - +20 points"
else
    echo "❌ New peer did not join 'ipchannel'"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if chaincode `ipcc` is deployed
echo "Checking if chaincode 'ipcc' is deployed..."
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "ipcc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'ipcc' is installed - +20 points"
else
    echo "❌ Chaincode 'ipcc' is not installed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if IP was registered correctly
echo "Querying registered IP 'ip1'..."
IP_OUTPUT=$(peer chaincode query -C ipchannel -n ipcc -c '{"Args":["VerifyOwnership","ip1"]}' 2>&1)

# Parse JSON using jq
IP_ID=$(echo "$IP_OUTPUT" | jq -r '.ipID' 2>/dev/null)
OWNER=$(echo "$IP_OUTPUT" | jq -r '.owner' 2>/dev/null)

# Step 4: Verify IP Registration
if [[ "$IP_ID" == "ip1" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ IP 'ip1' registered successfully - +20 points"
else
    echo "❌ IP 'ip1' registration failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Verify IP Ownership Transfer
if [[ "$OWNER" == "Bob" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ IP ownership transferred correctly - +20 points"
else
    echo "❌ IP ownership transfer failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Partial Credit Score
echo "Partial Credit: $SCORE"

```

