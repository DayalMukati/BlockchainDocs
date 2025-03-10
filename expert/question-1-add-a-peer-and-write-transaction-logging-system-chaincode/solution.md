# Solution

**Part 1: Adding a New Peer to Org1**

1.  **Update Configuration:**

    * In the `test-network/docker/docker-compose-test-net.yaml`, add configuration for the new peer (`peer1.org1.example.com`). It should include its container configuration, ports, environment variables, and volumes.

    Example Docker configuration for `peer1.org1.example.com`:

```go
peer1.org1.example.com:
    container_name: peer1.org1.example.com
    image: hyperledger/fabric-peer:2.4.9
    labels:
      service: hyperledger-fabric
    environment:
      - FABRIC_CFG_PATH=/etc/hyperledger/peercfg
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=false
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variables
      - CORE_PEER_ID=peer1.org1.example.com
      - CORE_PEER_ADDRESS=peer1.org1.example.com:8051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:8051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org1.example.com:8052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:8052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:8051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.example.com:8051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/fabric/msp
      - CORE_OPERATIONS_LISTENADDRESS=peer1.org1.example.com:9446
      - CORE_METRICS_PROVIDER=prometheus
      - CHAINCODE_AS_A_SERVICE_BUILDER_CONFIG={"peername":"peer1org1"}
      - CORE_CHAINCODE_EXECUTETIMEOUT=300s
    volumes:
        - ../organizations/peerOrganizations/org1.example.com/peers/peer1.org1.example.com:/etc/hyperledger/fabric
        - peer1.org1.example.com:/var/hyperledger/production
    working_dir: /root
    command: peer node start
    ports:
      - 8051:8051
      - 9446:9446
    networks:
      - test
```

### Steps to Deploy and Test:

1.  Go Inside lxp-transactionlogging-hlf/test-network and Start the test network if it’s not running:

    <pre class="language-bash"><code class="lang-bash">cd lxp-transactionlogging-hlf/test-network
    <strong>./network.sh up createChannel
    </strong></code></pre>
2.  Deploy the chaincode:

    ```bash
    ./network.sh deployCC -c mychannel -ccn transactionlog -ccp ../chaincode -ccl go
    ```
3. Create a tx log (Org1 user):

```bash
source ./scripts/setOrgPeerContext.sh 1
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
```

```
source ./scripts/setOrgPeerContext.sh 1  # Org1 context

peer chaincode invoke -o localhost:7050 \
    --ordererTLSHostnameOverride orderer.example.com \
    --tls --cafile $ORDERER_CA -C mychannel -n transactionlog \
    --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
    --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
    -c '{"Args":["LogTransaction","tx12345","Alice","Bob","100"]}'

```

```
Output: INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

4. Get the Tx Log (Org1 user):

```bash
source ./scripts/setOrgPeerContext.sh 1  # Org1 context

peer chaincode query -C mychannel -n transactionlog \
    -c '{"Args":["QueryTransaction","tx12345"]}'

```

```
INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```
