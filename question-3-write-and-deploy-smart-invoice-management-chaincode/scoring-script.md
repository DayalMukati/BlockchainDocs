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

# Step 1: Check if chaincode 'invoicecc' is deployed
CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "invoicecc"; then
    SCORE=$((SCORE + 20))
    echo "✅ Chaincode 'invoicecc' is deployed - +20 points"
else
    echo "❌ Chaincode 'invoicecc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 2: Check if invoice was created
INVOICE_DETAILS=$(peer chaincode query -C mychannel -n invoicecc -c '{"Args":["GetInvoiceDetails","inv1"]}' 2>&1)
if echo "$INVOICE_DETAILS" | jq -e '.invoiceID == "inv1"' > /dev/null; then
    SCORE=$((SCORE + 20))
    echo "✅ Invoice 'inv1' was successfully created - +20 points"
else
    echo "❌ Invoice creation failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 3: Check if the invoice was approved
INVOICE_STATUS=$(echo "$INVOICE_DETAILS" | jq -r '.status')
if [[ "$INVOICE_STATUS" == "Approved" || "$INVOICE_STATUS" == "Settled" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Invoice was approved - +20 points"
else
    echo "❌ Invoice approval failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 4: Check if invoice was settled
if [[ "$INVOICE_STATUS" == "Settled" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Invoice settlement confirmed - +20 points"
else
    echo "❌ Invoice settlement failed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Step 5: Verify final invoice transaction details
BUYER=$(echo "$INVOICE_DETAILS" | jq -r '.buyer')
SELLER=$(echo "$INVOICE_DETAILS" | jq -r '.seller')
AMOUNT=$(echo "$INVOICE_DETAILS" | jq -r '.amount')

if [[ "$BUYER" == "Alice" && "$SELLER" == "Bob" && "$AMOUNT" == "5000" ]]; then
    SCORE=$((SCORE + 20))
    echo "✅ Invoice details verified correctly - +20 points"
else
    echo "❌ Invoice details mismatch"
    echo "Partial Credit: $SCORE"
    exit 0
fi

# Display Final Score
echo "Partial Credit: $SCORE"

```

