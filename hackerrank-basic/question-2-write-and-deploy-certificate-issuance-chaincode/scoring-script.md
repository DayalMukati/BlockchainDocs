# Scoring Script

```shell
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

SCORE=0

# Query certificate details
echo "Querying cert1 details..."
CERT_OUTPUT=$(peer chaincode query -C mychannel -n certcc -c '{"Args":["GetCertificate","cert1"]}' 2>&1)
echo "$CERT_OUTPUT"

# Step 1: Check if cert1 exists
if echo "$CERT_OUTPUT" | grep -q '"certID": "cert1"'; then
    SCORE=$((SCORE + 40))
    echo "Certificate 'cert1' exists - +40 points"
else
    echo "Certificate 'cert1' not found"
fi

# Step 2: Check if cert1 is issued to the correct student
if echo "$CERT_OUTPUT" | grep -q '"studentName": "John Doe"'; then
    SCORE=$((SCORE + 30))
    echo "Certificate issued to 'John Doe' - +30 points"
elif echo "$CERT_OUTPUT" | grep -q '"studentName":'; then
    SCORE=$((SCORE + 10))
    echo "Certificate exists but issued to the wrong student - +10 points"
else
    echo "Certificate does not contain a valid student name"
fi

# Step 3: Check if course name is correct
if echo "$CERT_OUTPUT" | grep -q '"courseName": "Blockchain Basics"'; then
    SCORE=$((SCORE + 30))
    echo "Correct course name 'Blockchain Basics' - +30 points"
else
    echo "Incorrect or missing course name"
fi

# Display the final score
echo "Final Score: $SCORE"

```
