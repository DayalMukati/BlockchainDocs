# Scoring Script

```
#!/usr/bin/env bash

echo "Partial Credit: 0"

cd ./test-network
source ./scripts/setOrgPeerContext.sh 1

if ! command -v jq &> /dev/null; then
    echo "Error: jq is not installed. Install it using: sudo apt install jq"
    exit 1
fi

SCORE=0

CHAINCODE_INFO=$(peer lifecycle chaincode queryinstalled 2>&1)
if echo "$CHAINCODE_INFO" | grep -q "paymentscc"; then
    SCORE=$((SCORE + 30))
    echo "✅ Chaincode 'paymentscc' is deployed - +30 points"
else
    echo "❌ Chaincode 'paymentscc' is not deployed"
    echo "Partial Credit: $SCORE"
    exit 0
fi

BALANCE_ALICE=$(peer chaincode query -C mychannel -n paymentscc -c '{"Args":["GetBalance","account1"]}' | jq -r '.')
BALANCE_BOB=$(peer chaincode query -C mychannel -n paymentscc -c '{"Args":["GetBalance","account2"]}' | jq -r '.')

if [[ "$BALANCE_ALICE" == "800" && "$BALANCE_BOB" == "700" ]]; then
    SCORE=$((SCORE + 70))
    echo "✅ Transaction successful - +70 points"
else
    echo "❌ Transaction failed"
fi

echo "Partial Credit: $SCORE"

```

