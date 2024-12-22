# ▶️ Chaincode Execution

1.  Before interaction with chaincode, we need to set up the environment path and variables to ensure smooth execution.&#x20;

    \
    Let’s open the terminal, navigate to the fabric-samples/test-network directory and enter the commands below:

    &#x20;       The first command sets the path to include the hyperledger fabrics binaries directory.



```
export PATH=${PWD}/../bin:${PWD}:$PATH

export CHANNEL_NAME=mychannel 

source ./scripts/setOrgPeerContext.sh 1

```

2. Run the command below to initialize the chaincode

{% code overflow="wrap" %}
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n asset $PEER_CONN_PARAMS --isInit -c '{"function":"InitLedger","Args":[]}'
```
{% endcode %}

3. Run the command below to get the assets list from the ledger.

{% code overflow="wrap" %}
```
peer chaincode query -C $CHANNEL_NAME -n asset -c '{"Args":["GetAllAssets"]}'
```
{% endcode %}

4. Run the command below to create a new asset.

{% code overflow="wrap" %}
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n asset $PEER_CONN_PARAMS -c '{"function":"CreateAsset","Args":["asset11","red","8", "Ron", "500"]}'
```
{% endcode %}

5. Run the command below to read the asset.

{% code overflow="wrap" %}
```
source ./scripts/setOrgPeerContext.sh 2

peer chaincode query -C $CHANNEL_NAME -n asset -c '{"Args":["ReadAsset","asset11"]}'

```
{% endcode %}

Play around with it.
