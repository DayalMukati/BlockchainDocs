# Solution

## Solution:

**Smart Contract: `supplychain.go`**

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/hyperledger/fabric-contract-api-go/contractapi"
)

type SupplyChainContract struct {
	contractapi.Contract
}

type Product struct {
	ProductID   string `json:"productID"`
	ProductName string `json:"productName"`
	Owner       string `json:"owner"`
	Status      string `json:"status"`
}

func (s *SupplyChainContract) CreateProduct(ctx contractapi.TransactionContextInterface, productID string, productName string, owner string) error {
	product := Product{
		ProductID:   productID,
		ProductName: productName,
		Owner:       owner,
		Status:      "created",
	}

	productAsBytes, err := json.Marshal(product)
	if err != nil {
		return fmt.Errorf("failed to marshal product: %s", err.Error())
	}

	return ctx.GetStub().PutState(productID, productAsBytes)
}

func (s *SupplyChainContract) TransferProduct(ctx contractapi.TransactionContextInterface, productID string, newOwner string) error {
	productAsBytes, err := ctx.GetStub().GetState(productID)
	if err != nil {
		return fmt.Errorf("failed to get product: %s", err.Error())
	} else if productAsBytes == nil {
		return fmt.Errorf("product %s does not exist", productID)
	}

	product := new(Product)
	err = json.Unmarshal(productAsBytes, product)
	if err != nil {
		return fmt.Errorf("failed to unmarshal product: %s", err.Error())
	}

	product.Owner = newOwner

	productAsBytes, err = json.Marshal(product)
	if err != nil {
		return fmt.Errorf("failed to marshal product: %s", err.Error())
	}

	return ctx.GetStub().PutState(productID, productAsBytes)
}

func (s *SupplyChainContract) GetProduct(ctx contractapi.TransactionContextInterface, productID string) (*Product, error) {
	productAsBytes, err := ctx.GetStub().GetState(productID)
	if err != nil {
		return nil, fmt.Errorf("failed to get product: %s", err.Error())
	} else if productAsBytes == nil {
		return nil, fmt.Errorf("product %s does not exist", productID)
	}

	product := new(Product)
	err = json.Unmarshal(productAsBytes, product)
	if err != nil {
		return nil, fmt.Errorf("failed to unmarshal product: %s", err.Error())
	}

	return product, nil
}

func main() {
	chaincode, err := contractapi.NewChaincode(new(SupplyChainContract))
	if err != nil {
		fmt.Printf("Error creating supply chain chaincode: %s", err.Error())
		return
	}

	if err := chaincode.Start(); err != nil {
		fmt.Printf("Error starting supply chain chaincode: %s", err.Error())
	}
}
```

#### Steps to Deploy and Test:

1.  Start the test network if it’s not running:

    ```bash
    ./network.sh up createChannel
    ```
2.  Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn supplychain -ccp ../chaincode -ccl go
    ```
3. Create a product:

```bash
source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

<pre><code><strong>peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C mychannel -n supplychain --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["CreateProduct","prod1","CoffeeBeans","SupplierA"]}'
</strong></code></pre>

```
Output: INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

4. Transfer the product:

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C mychannel -n supplychain --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA -c '{"Args":["TransferProduct","prod1","DistributorA"]}'

```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

5. Query the product:

```bash
peer chaincode query -C mychannel -n supplychain -c '{"Args":["GetProduct","prod1"]}'
```

```
{"productID":"prod1","productName":"CoffeeBeans","owner":"DistributorA","status":"created"}
```
