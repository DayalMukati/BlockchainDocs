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

# Step 1: Check if chaincode 'paymentcc' is deployed
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "paymentcc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'paymentcc' is deployed - +20 points"
else
    echo "❌ Chaincode 'paymentcc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if payment was initiated
PAYMENT_DETAILS=$(peer chaincode query -C paymentschannel -n paymentcc -c '{"Args":["GetPaymentDetails","tx1"]}' 2>&1)
if echo "$PAYMENT_DETAILS" | jq -e '.txID == "tx1"' > /dev/null; then
    SCORE=$((SCORE + 20))
    echo "✅ Payment transaction 'tx1' was successfully initiated - +20 points"
else
    echo "❌ Payment initiation failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if the payment was approved
PAYMENT_STATUS=$(echo "$PAYMENT_DETAILS" | jq -r '.status')
if [[ "$PAYMENT_STATUS" == "Approved" || "$PAYMENT_STATUS" == "Settled" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Payment transaction was approved - +20 points"
else
    echo "❌ Payment approval failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if payment was settled
if [[ "$PAYMENT_STATUS" == "Settled" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Payment settlement confirmed - +20 points"
else
    echo "❌ Payment settlement failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Verify final transaction details
SENDER=$(echo "$PAYMENT_DETAILS" | jq -r '.sender')
RECEIVER=$(echo "$PAYMENT_DETAILS" | jq -r '.receiver')
AMOUNT=$(echo "$PAYMENT_DETAILS" | jq -r '.amount')

if [[ "$SENDER" == "BankA" && "$RECEIVER" == "BankB" && "$AMOUNT" == "10000" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Transaction details verified correctly - +20 points"
else
    echo "❌ Transaction details mismatch"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Final Score
echo "Partial Credit: $SCORE"

```

