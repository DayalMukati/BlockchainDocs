# Question 1: Decentralized Voting System

**Problem Statement:**

You are tasked with developing a decentralized voting system using Hyperledger Fabric. The objective is to create a smart contract (chaincode) that manages the election process, ensuring security, transparency, and voter anonymity.

**Requirements:**

1. **Smart Contract Functionality:**
   * **RegisterVoter(voterID, voterDetails):** Enroll a new voter with a unique identifier and associated details (e.g., name, eligibility status).
   * **CreateElection(electionID, electionDetails):** Initiate a new election with specific parameters (e.g., list of candidates, start and end times).
   * **CastVote(voterID, electionID, candidateID):** Allow a registered voter to cast a vote for a chosen candidate in a specific election.
   * **TallyVotes(electionID):** Count and return the votes for each candidate in the specified election.
   * **GetElectionResults(electionID):** Retrieve and display the final results of the election.
2. **Deployment Environment:**
   * Utilize a pre-configured Hyperledger Fabric test network with the following setup:
     * Two organizations (Org1 and Org2), each with one peer.
     * A default channel named `mychannel`.
     * No pre-existing data on the ledger.
3. **Implementation Details:**
   * Write the smart contract in Go (Golang).
   * Deploy the chaincode `votingcc` on the `mychannel` channel using the test network.
   * Test the smart contract by performing the following actions:
     * Register a voter with ID `voter1` and details `{Name: "John Doe", Eligibility: true}`.
     * Create an election with ID `election1` and details `{Title: "Board Election", Candidates: ["Alice", "Bob"], StartTime: "2025-03-01T08:00:00Z", EndTime: "2025-03-01T20:00:00Z"}`.
     * Cast a vote from `voter1` for candidate `Alice` in `election1`.
     * Tally the votes for `election1` after the election ends.
     * Retrieve and display the results of `election1`.

**Notes:**

* The Hyperledger Fabric test network can be initiated using the command `./network.sh up createChannel` with the default channel `mychannel`.
* Peers from Org1 and Org2 are already joined to the channel.
* The chaincode environment is pre-configured in the `chaincode` directory.

**Deployment Steps:**

1. **Deploy the Chaincode:**
   *   Navigate to the test network directory:

       ```bash
       cd fabric-samples/test-network
       ```
   *   Deploy the chaincode:

       ```bash
       sudo ./network.sh deployCC -c mychannel -ccn votingcc -ccp ../chaincode -ccl go
       ```
2. **Set Organization Context:**
   *   Set the context for Org1:

       <pre class="language-bash"><code class="lang-bash">export FABRIC_CFG_PATH=${PWD}/configtx
       <strong>source ./scripts/setOrgPeerContext.sh 1
       </strong></code></pre>
3. **Interact with the Chaincode:**
   *   **Register Voter:**

       ```bash
       peer chaincode invoke -o localhost:7050 \
         --ordererTLSHostnameOverride orderer.example.com \
         --tls --cafile $ORDERER_CA -C mychannel -n votingcc \
         --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
         --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
         -c '{"Args":["RegisterVoter","voter1","{\"Name\":\"John Doe\",\"Eligibility\":true}"]}'
       ```
   *   **Create Election:**

       ```bash
       peer chaincode invoke -o localhost:7050 \
         --ordererTLSHostnameOverride orderer.example.com \
         --tls --cafile $ORDERER_CA -C mychannel -n votingcc \
         --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
         --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
         -c '{"Args":["CreateElection","election1","Board Election","[\"Alice\",\"Bob\"]","2025-03-01T08:00:00Z","2025-06-01T20:00:00Z"]}'

       ```
   *   **Cast Vote:**

       ```bash
       peer chaincode invoke -o localhost:7050 \
         --ordererTLSHostnameOverride orderer.example.com \
         --tls --cafile $ORDERER_CA -C mychannel -n votingcc \
         --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
         --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
         -c '{"Args":["CastVote","voter1","election1","Alice"]}'
       ```
   *   **Tally Votes:**

       ```bash
       peer chaincode invoke -o localhost:7050 \
         --ordererTLSHostnameOverride orderer.example.com \
         --tls --cafile $ORDERER_CA -C mychannel -n votingcc \
         --peerAddresses localhost:7051 --tlsRootCertFiles $PEER0_ORG1_CA \
         --peerAddresses localhost:9051 --tlsRootCertFiles $PEER0_ORG2_CA \
         -c '{"Args":["TallyVotes","election1"]}'
       ```
   *   **Get Election Results:**

       ```bash
       peer chaincode query -C mychannel -n votingcc -c '{"Args":["GetElectionResults","election1"]}'
       ```

By completing this task, you will demonstrate the ability to develop, deploy, and interact with a decentralized voting system on a Hyperledger Fabric network.
