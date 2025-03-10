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

SCORE=0

# Step 1: Check if `peer1.org1.example.com` is running
echo "Checking if peer1.org1.example.com is running..."
PEER1_STATUS=$(docker ps | grep peer1.org1.example.com)
if [ -n "$PEER1_STATUS" ]; then
    SCORE=$((SCORE + 20))
    echo "✅ New peer 'peer1.org1.example.com' is running - +20 points"
else
    echo "❌ New peer 'peer1.org1.example.com' is not running"
fi

# Step 2: Check if `peer1.org1.example.com` joined `ipchannel`
echo "Checking if peer1.org1.example.com joined 'ipchannel'..."
PEER_JOINED=$(peer channel list | grep ipchannel)
if echo "$PEER_JOINED" | grep -q "ipchannel"; then
    SCORE=$((SCORE + 20))
    echo "✅ New peer joined 'ipchannel' - +20 points"
else
    echo "❌ New peer did not join 'ipchannel'"
fi

# Step 3: Check if chaincode `ipcc` is deployed
echo "Checking if chaincode 'ipcc' is deployed..."
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled | grep ipcc)
if [ -n "$CHAINCODE_INFO" ]; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'ipcc' is installed - +20 points"
else
    echo "❌ Chaincode 'ipcc' is not installed"
fi

# Step 4: Check if IP was registered correctly
echo "Querying registered IP 'ip1'..."
IP_OUTPUT=$(peer chaincode query -C ipchannel -n ipcc -c '{"Args":["VerifyOwnership","ip1"]}' 2>&1)
if echo "$IP_OUTPUT" | grep -q '"ipID": "ip1"'; then
    SCORE=$((SCORE + 10))
    echo "✅ IP 'ip1' registered successfully - +10 points"
else
    echo "❌ IP 'ip1' registration failed"
fi

# Step 5: Check if IP ownership was transferred
echo "Checking if IP ownership was transferred..."
if echo "$IP_OUTPUT" | grep -q '"owner": "Bob Studios"'; then
    SCORE=$((SCORE + 10))
    echo "✅ IP ownership transferred correctly - +10 points"
else
    echo "❌ IP ownership transfer failed"
fi

# Step 6: Check if IP ownership verification is correct
echo "Checking if IP ownership verification works..."
if echo "$IP_OUTPUT" | grep -q '"owner": "Bob Studios"'; then
    SCORE=$((SCORE + 20))
    echo "✅ Ownership verification successful - +20 points"
else
    echo "❌ Ownership verification failed"
fi

# Display final score
echo "Final Score: $SCORE"

```

