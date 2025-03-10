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

# Check if the chaincode 'votingcc' is installed
CHAINCODE_CHECK=$(peer lifecycle chaincode queryinstalled 2>&1)

if echo "$CHAINCODE_CHECK" | grep -q "votingcc"; then
    SCORE=$((SCORE + 40))
    echo "Voting chaincode 'votingcc' is installed - +40 points"
else
    echo "Voting chaincode 'votingcc' is not installed"
fi

# Query election results
RESULTS_OUTPUT=$(peer chaincode query -C mychannel -n votingcc -c '{"Args":["GetElectionResults","election1"]}' 2>&1)

# Ensure jq is installed
if ! command -v jq &> /dev/null; then
   echo "Error: jq is not installed. Please install it using: sudo apt install jq"
   exit 1
fi

# Parse JSON values using jq
VOTE_COUNT_ALICE=$(echo "$RESULTS_OUTPUT" | jq -r '.Alice' 2>/dev/null)

# Debugging: Print extracted values
echo "Extracted votes for Alice: $VOTE_COUNT_ALICE"

# Step 2: Check if election results exist
if [[ -n "$RESULTS_OUTPUT" && "$RESULTS_OUTPUT" != "null" ]]; then
   SCORE=$((SCORE + 30))
   echo "Election results exist - +30 points"
else
   echo "Election results not found"
fi

# Step 3: Check if Alice received votes
if [[ "$VOTE_COUNT_ALICE" -ge 1 ]]; then
   SCORE=$((SCORE + 30))
   echo "Vote successfully cast for 'Alice' - +30 points"
else
   echo "Vote for 'Alice' not found"
fi

# Print Final Score
echo "Partial Credit: $SCORE"

```

