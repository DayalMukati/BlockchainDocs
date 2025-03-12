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

# Step 1: Check if chaincode 'creditscorecc' is deployed
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "creditscorecc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'creditscorecc' is deployed - +20 points"
else
    echo "❌ Chaincode 'creditscorecc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if Alice is registered
USER_DETAILS=$(peer chaincode query -C mychannel -n creditscorecc -c '{"Args":["GetUserCreditHistory","user1"]}' 2>&1)
if echo "$USER_DETAILS" | jq -e '.userID == "user1"' > /dev/null; then
    SCORE=$((SCORE + 10))
    echo "✅ Alice (user1) is registered - +10 points"
else
    echo "❌ Alice is not registered"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if Alice’s credit score was updated to 800
CREDIT_SCORE=$(echo "$USER_DETAILS" | jq -r '.creditScore')
if [[ "$CREDIT_SCORE" == "800" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Alice's credit score updated to 800 - +20 points"
else
    echo "❌ Credit score update failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if the loan was issued
LOAN_DETAILS=$(peer chaincode query -C mychannel -n creditscorecc -c '{"Args":["GetLoan","loan1"]}' 2>&1)
if echo "$LOAN_DETAILS" | jq -e '.loanID == "loan1"' > /dev/null; then
    SCORE=$((SCORE + 20))
    echo "✅ Loan of $2000 was issued to Alice - +20 points"
else
    echo "❌ Loan issuance failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Verify loan repayment of $500 (Balance should be $1500)
LOAN_BALANCE=$(echo "$LOAN_DETAILS" | jq -r '.balance')
if [[ "$LOAN_BALANCE" == "1500" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Loan repayment recorded correctly - +20 points"
else
    echo "❌ Loan repayment verification failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 6: Confirm credit history retrieval
if echo "$USER_DETAILS" | jq -e '.userID' > /dev/null; then
    SCORE=$((SCORE + 10))
    echo "✅ Credit history retrieved successfully - +10 points"
else
    echo "❌ Failed to retrieve credit history"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Final Score
echo "Partial Credit: $SCORE"

```

