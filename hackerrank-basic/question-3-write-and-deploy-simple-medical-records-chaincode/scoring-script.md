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

# Fetch patient record details from chaincode query
PATIENT_OUTPUT=$(peer chaincode query -C mychannel -n medicalcc -c '{"Args":["GetPatientDetails","patient1"]}' 2>&1)

# Ensure jq is installed
if ! command -v jq &> /dev/null; then
   echo "Error: jq is not installed. Please install it using: sudo apt install jq"
   exit 1
fi

# Parse JSON values using jq
PATIENT_ID=$(echo "$PATIENT_OUTPUT" | jq -r '.patientID' 2>/dev/null)
PATIENT_NAME=$(echo "$PATIENT_OUTPUT" | jq -r '.name' 2>/dev/null)
PATIENT_AGE=$(echo "$PATIENT_OUTPUT" | jq -r '.age' 2>/dev/null)
MEDICAL_HISTORY=$(echo "$PATIENT_OUTPUT" | jq -r '.medicalHistory' 2>/dev/null)

# Debugging: Print extracted values
echo "Extracted patientID: $PATIENT_ID"
echo "Extracted name: $PATIENT_NAME"
echo "Extracted age: $PATIENT_AGE"
echo "Extracted medical history: $MEDICAL_HISTORY"

# Step 1: Check if patient1 exists
if [[ "$PATIENT_ID" == "patient1" ]]; then
   SCORE=$((SCORE + 30))
   echo "Patient record 'patient1' exists - +30 points"
else
   echo "Patient record 'patient1' not found"
fi

# Step 2: Check if the patient name is correct
if [[ "$PATIENT_NAME" == "Alice Johnson" ]]; then
   SCORE=$((SCORE + 20))
   echo "Correct patient name 'Alice Johnson' - +20 points"
else
   echo "Incorrect or missing patient name"
fi

# Step 3: Check if the age is correct
if [[ "$PATIENT_AGE" == "30" ]]; then
   SCORE=$((SCORE + 20))
   echo "Correct patient age '30' - +20 points"
else
   echo "Incorrect or missing age"
fi

# Step 4: Check if medical history was updated
if [[ "$MEDICAL_HISTORY" == *"Diagnosed with Hypertension"* ]]; then
   SCORE=$((SCORE + 30))
   echo "Medical history correctly updated with 'Diagnosed with Hypertension' - +30 points"
else
   echo "Medical history update failed"
fi

# Print Final Score
echo "Partial Credit: $SCORE"

```

