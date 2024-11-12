# 👨‍💻 Test Network

The test network is provided for learning about Fabric by running nodes on your local machine. Developers can use the network to test their smart contracts and applications. The network is meant to be used only as a tool for education and testing. In general, modifications to the scripts are discouraged and could break the network. It is based on a limited configuration that should not be used as a template for deploying a production network:

* It includes two peer organizations and an ordering organization.
* For simplicity, a single node Raft ordering service is configured.
* To reduce complexity, a TLS Certificate Authority (CA) is not deployed. All certificates are issued by the root CAs.
* The sample network deploys a Fabric network with Docker Compose. Because the nodes are isolated within a Docker Compose network, the test network is not configured to connect to other running Fabric nodes.

Each node and user that interacts with a Fabric network need to belong to an organization in order to participate in the network. The test network includes two peer organizations, Org1 and Org2. It also includes a single orderer organization that maintains the ordering service of the network.

​

[Peers](https://hyperledger-fabric.readthedocs.io/en/release-2.2/peers/peers.html)

are the fundamental components of any Fabric network. Peers store the blockchain ledger and validate transactions before they are committed to the ledger. Peers run the smart contracts that contain the business logic that is used to manage the assets on the blockchain ledger.

Every peer in the network needs to belong to an organization. In the test network, each organization operates one peer each, `peer0.org1.example.com` and `peer0.org2.example.com`.

Every Fabric network also includes an

[ordering service](https://hyperledger-fabric.readthedocs.io/en/release-2.2/orderer/ordering\_service.html)

. While peers validate transactions and add blocks of transactions to the blockchain ledger, they do not decide on the order of transactions or include them in new blocks. On a distributed network, peers may be running far away from each other and not have a common view of when a transaction was created. Coming to a consensus on the order of transactions is a costly process that would create overhead for the peers.

An ordering service allows peers to focus on validating transactions and committing them to the ledger. After ordering nodes receive endorsed transactions from clients, they come to a consensus on the order of transactions and then add them to blocks. The blocks are then distributed to peer nodes, which add the blocks to the blockchain ledger.

The sample network uses a single node Raft ordering service that is operated by the orderer organization. You can see the ordering node running on your machine as `orderer.example.com`. While the test network only uses a single node ordering service, a production network would have multiple ordering nodes, operated by one or multiple orderer organizations. The different ordering nodes would use the Raft consensus algorithm to come to an agreement on the order of transactions across the network.

We will have a fabric channel for transactions between Org1 and Org2. Channels are a private layer of communication between specific network members. Channels can be used only by organizations that are invited to the channel, and are invisible to other members of the network. Each channel has a separate blockchain ledger. Organizations that have been invited “join” their peers to the channel to store the channel ledger and validate the transactions on the channel.
