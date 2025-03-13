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

# Step 1: Check if chaincode 'paymentscc' is deployed
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "paymentscc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'paymentscc' is deployed - +20 points"
else
    echo "❌ Chaincode 'paymentscc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if payment was initiated
PAYMENT_DETAILS=$(peer chaincode query -C mychannel -n paymentscc -c '{"Args":["GetPaymentDetails","payment1"]}' 2>&1)
if echo "$PAYMENT_DETAILS" | jq -e '.paymentID == "payment1"' > /dev/null; then
    SCORE=$((SCORE + 20))
    echo "✅ Payment 'payment1' was successfully initiated - +20 points"
else
    echo "❌ Payment initiation failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if the payment was approved
PAYMENT_STATUS=$(echo "$PAYMENT_DETAILS" | jq -r '.status')
if [[ "$PAYMENT_STATUS" == "Approved" || "$PAYMENT_STATUS" == "Settled" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Payment was approved - +20 points"
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

# Step 5: Verify final payment transaction details
SENDER_BANK=$(echo "$PAYMENT_DETAILS" | jq -r '.senderBank')
RECEIVER_BANK=$(echo "$PAYMENT_DETAILS" | jq -r '.receiverBank')
AMOUNT=$(echo "$PAYMENT_DETAILS" | jq -r '.amount')

if [[ "$SENDER_BANK" == "BankA" && "$RECEIVER_BANK" == "BankB" && "$AMOUNT" == "5000" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Payment details verified correctly - +20 points"
else
    echo "❌ Payment details mismatch"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Final Score
echo "Partial Credit: $SCORE"

```

