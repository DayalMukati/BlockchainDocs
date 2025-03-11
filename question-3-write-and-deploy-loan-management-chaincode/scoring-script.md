# Scoring Script

```
#!/usr/bin/env bash

echo "Partial Credit: 0"

cd ./test-network
source ./scripts/setOrgPeerContext.sh 1

# Ensure jq is installed
if ! command -v jq &> /dev/null; then
    echo "Error: jq is not installed. Install it using: sudo apt install jq"
    exit 1
fi

SCORE=0

# Step 1: Check if chaincode 'loancc' is deployed
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "loancc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'loancc' is deployed - +20 points"
else
    echo "❌ Chaincode 'loancc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if Alice's loan application exists
LOAN_DETAILS=$(peer chaincode query -C mychannel -n loancc -c '{"Args":["GetLoan","loan1"]}' 2>&1)
if echo "$LOAN_DETAILS" | jq -e '.loanID == "loan1"' > /dev/null; then
    SCORE=$((SCORE + 20))
    echo "✅ Alice's loan application exists - +20 points"
else
    echo "❌ Loan application not found"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if the loan was approved
LOAN_STATUS=$(echo "$LOAN_DETAILS" | jq -r '.status')
if [[ "$LOAN_STATUS" == "Approved" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Loan is approved - +20 points"
else
    echo "❌ Loan is not approved"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if repayment transaction was executed
LOAN_BALANCE=$(echo "$LOAN_DETAILS" | jq -r '.balance')
if [[ "$LOAN_BALANCE" == "500" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Loan repayment of $500 recorded correctly - +20 points"
else
    echo "❌ Loan repayment failed or incorrect balance"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Final confirmation of loan details
if [[ "$LOAN_BALANCE" == "500" && "$LOAN_STATUS" == "Approved" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Loan system working correctly - +20 points"
else
    echo "❌ Loan verification failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Final Score
echo "Partial Credit: $SCORE"

```

