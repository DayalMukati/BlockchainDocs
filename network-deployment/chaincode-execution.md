# ▶️ Chaincode Execution

1. This chaincode fabcar, has to be initialized before executing other transactions.

```
source ./scripts/setPeerConnectionParam.sh 1 2

source ./scripts/setOrgPeerContext.sh 1

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n fabcar $PEER_CONN_PARAMS --isInit -c '{"function":"initLedger","Args":[]}'

```

2. Let's create a new car

{% code overflow="wrap" %}
```
source ./scripts/setOrgPeerContext.sh 1

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n fabcar $PEER_CONN_PARAMS -c '{"function":"CreateCar","Args":["CAR11","Tata","Safari", "Red", "Ron"]}'
```
{% endcode %}

3. Query the Status of the state after init.

{% code overflow="wrap" %}
```
peer chaincode query -C $CHANNEL_NAME -n fabcar -c '{"Args":["queryAllCars"]}'
```
{% endcode %}

4. Change the ownership of CAR11 to John:

{% code overflow="wrap" %}
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n fabcar $PEER_CONN_PARAMS -c '{"function":"changeCarOwner","Args":["CAR11","John"]}'
```
{% endcode %}

5. As Org2 query to check the status as ownership of CAR11

{% code overflow="wrap" %}
```
source ./scripts/setOrgPeerContext.sh 2

peer chaincode query -C $CHANNEL_NAME -n fabcar -c '{"Args":["queryCar","CAR11"]}'

```
{% endcode %}

Play around with it.
