# Scoring Script

```
#!/usr/bin/env bash

# Start with Partial Credit: 0
echo "Partial Credit: 0"

# Navigate to test network
cd ./test-network

# Set environment for Org1
source ./scripts/setOrgPeerContext.sh 1

# Ensure jq is installed
if ! command -v jq &> /dev/null; then
    echo "Error: jq is not installed. Install it using: sudo apt install jq"
    exit 1
fi

SCORE=0

# Step 1: Check if chaincode 'studentcc' is deployed
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "studentcc"; then
    SCORE=$((SCORE + 30))
    echo "✅ Chaincode 'studentcc' is deployed - +30 points"
else
    echo "❌ Chaincode 'studentcc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Verify student registration
STUDENT_OUTPUT=$(peer chaincode query -C mychannel -n studentcc -c '{"Args":["GetStudent","student1"]}' 2>&1)

# Extract data using jq
STUDENT_ID=$(echo "$STUDENT_OUTPUT" | jq -r '.studentID')
STUDENT_NAME=$(echo "$STUDENT_OUTPUT" | jq -r '.name')
STUDENT_COURSE=$(echo "$STUDENT_OUTPUT" | jq -r '.course')

# Step 3: Check if the student exists
if [[ "$STUDENT_ID" == "student1" ]]; then
    SCORE=$((SCORE + 30))
    echo "✅ Student 'student1' exists - +30 points"
else
    echo "❌ Student record not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if student name is correct
if [[ "$STUDENT_NAME" == "John Doe" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Student name is 'John Doe' - +20 points"
else
    echo "❌ Incorrect student name"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Verify if course update was successful
if [[ "$STUDENT_COURSE" == "Hyperledger Fabric" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Course updated to 'Hyperledger Fabric' - +20 points"
else
    echo "❌ Course update failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Final Score
echo "Partial Credit: $SCORE"

```

