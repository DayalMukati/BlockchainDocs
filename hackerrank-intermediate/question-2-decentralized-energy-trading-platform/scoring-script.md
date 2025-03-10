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

# Step 1: Check if chaincode 'energycc' is installed
echo "Checking if chaincode 'energycc' is installed..."
CHAINCODE_CHECK=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_CHECK" | grep -q "energycc"; then
    SCORE=$((SCORE + 30))
    echo "Energy trading chaincode 'energycc' is installed - +30 points"
else
    echo "Energy trading chaincode 'energycc' is not installed"
    exit 0
fi

# Step 2: Query user balances and energy units
PRODUCER_DETAILS=$(peer chaincode query -C mychannel -n energycc -c '{"Args":["GetUserDetails","producer1"]}' 2>&1)
CONSUMER_DETAILS=$(peer chaincode query -C mychannel -n energycc -c '{"Args":["GetUserDetails","consumer1"]}' 2>&1)

# Check if producer generated energy
if echo "$PRODUCER_DETAILS" | grep -q '"energyUnits": 100'; then
    SCORE=$((SCORE + 20))
    echo "Producer successfully generated energy - +20 points"
fi

# Verify energy transfer
if echo "$CONSUMER_DETAILS" | grep -q '"energyUnits": 50' && echo "$PRODUCER_DETAILS" | grep -q '"balance": 1100'; then
    SCORE=$((SCORE + 20))
    echo "Energy successfully traded - +20 points"
fi

echo "Partial Credit: $SCORE"

```

