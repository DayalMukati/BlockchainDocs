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

# Fetch land record details from chaincode query
LAND_OUTPUT=$(peer chaincode query -C mychannel -n landcc -c '{"Args":["GetLandDetails","land1"]}' 2>&1)

# Ensure jq is installed
if ! command -v jq &> /dev/null; then
   echo "Error: jq is not installed. Please install it using: sudo apt install jq"
   exit 1
fi

# Parse JSON values using jq
LAND_ID=$(echo "$LAND_OUTPUT" | jq -r '.landID' 2>/dev/null)
OWNER_NAME=$(echo "$LAND_OUTPUT" | jq -r '.ownerName' 2>/dev/null)
LOCATION=$(echo "$LAND_OUTPUT" | jq -r '.location' 2>/dev/null)

# Debugging: Print extracted values
echo "Extracted landID: $LAND_ID"
echo "Extracted ownerName: $OWNER_NAME"
echo "Extracted location: $LOCATION"

# Step 1: Check if land1 exists
if [[ "$LAND_ID" == "land1" ]]; then
   SCORE=$((SCORE + 40))
   echo "Land record 'land1' exists - +40 points"
else
   echo "Land record 'land1' not found"
fi

# Step 2: Check if land1 is assigned to the correct owner
if [[ "$OWNER_NAME" == "Charlie" ]]; then
   SCORE=$((SCORE + 30))
   echo "Land record belongs to 'Charlie' - +30 points"
elif [[ -n "$OWNER_NAME" ]]; then
   SCORE=$((SCORE + 10))
   echo "Land record exists but assigned to the wrong owner - +10 points"
else
   echo "Land record does not contain a valid owner"
fi

# Step 3: Check if location is correct
if [[ "$LOCATION" == "NYC" ]]; then
   SCORE=$((SCORE + 30))
   echo "Correct location 'NYC' recorded - +30 points"
else
   echo "Incorrect or missing location information"
fi

# Print Final Score
echo "Partial Credit: $SCORE"

```

