---
cover: https://www.hyperledger.org/wp-content/uploads/2018/06/HL_Default_fabric.png
coverY: 0
---

# 💡 Hyperledger Fabric

Hyperledger is an open-source collaborative effort created to advance cross-industry blockchain technologies. It is a global collaboration including leaders in banking, finance, Internet of Things, manufacturing, supply chain, and technology. The Linux Foundation hosts Hyperledger under the foundation.

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2F7BM6hQhwX0ZoT21GVSmQ%2FHL\_Greenhouse\_Current.svg?alt=media\&token=2805bc9b-ee7e-498c-9397-818c8d5ff44c)



* CONSENSUS - Participants will collectively agree that each transaction is valid and the order of the transaction in relation to others. (ex: 2 of 6 or 3 of 6)
* PROVENANCE - Participants know the history of the asset, for example, how its ownership has changed over time, the lifecycle, the history of the asset.
* IMMUTABILITY - No participant can tamper with a transaction once it's agreed.
* FINALITY - Once a transaction is committed, it cannot be reversed, in other words, the data cannot be rolled back to the previous state. If a transaction was in error then a NEW transaction must be used to reverse the error, with both transactions visible.

#### Hyperledger Fabric Architectural Model <a href="#hyperledger-fabric-architectural-model" id="hyperledger-fabric-architectural-model"></a>

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2FuYuzpp7ANbKYuSd4vYzR%2FHyperledger%20Fabric%20Architectural%20Model.png?alt=media\&token=bc563491-c9a4-4e94-97d5-fbc22e779f23)

#### Hyperledger Fabric (HLF) Components <a href="#hyperledger-fabric-hlf-components" id="hyperledger-fabric-hlf-components"></a>

![](https://camo.githubusercontent.com/d35e7aa27c7580a872984f442f2975f1a78548521e02eb0e768def1986a671d2/68747470733a2f2f616c6578616e647265626172726f732e636f6d2f676c6f62616c2f68797065726c65646765722f436f6d706f6e656e74732e706e673f616c743d68797065726c65646765722d636f6d706f6e656e7473)

#### Hyperledger Fabric Architecture <a href="#hyperledger-fabric-architecture" id="hyperledger-fabric-architecture"></a>

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2FNyLhy2erFxsL3EvicOGF%2FArchitecture2.png?alt=media\&token=89f4536b-f9c5-4990-b804-0c8817bf7486)

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2FfsqoSL1CG7gC2Fp8QYOR%2Farchitecture2%20\(1\).png?alt=media\&token=c1cbd5b9-3119-4d5a-bd72-60be828e4c57)

#### Application interaction with HLF <a href="#application-interaction-with-hlf" id="application-interaction-with-hlf"></a>

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2F10dqjM4aB8YEG2BG8KDl%2FApp-hlf.png?alt=media\&token=4d1597fa-5cd2-4c4c-967b-21554d8c8bb6)

#### Requirements for an enterprise blockchain use case <a href="#requirements-for-an-enterprise-blockchain-use-case" id="requirements-for-an-enterprise-blockchain-use-case"></a>

Blockchains for business generally prioritize – Assets over cryptocurrency; Identity over anonymity; Selective endorsement over proof of work.

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2Fs7Pw5s7UXatmFOfasbEE%2Frequirements-blockchain-use-case.png?alt=media\&token=286d6f66-a97c-4530-b24b-7408b8eb0c82)

**Assets** are anything of value, on the blockchain, these are represented digitally using a pre-agreed format.

**Transactions** change the state of an asset and are provably recorded on the blockchain ( e.g.transfer ownership, change colour) they are underpinned by smart contracts, verifiable business rules that cause the asset to change state

**Identity** is used to ensure business networks are kept private and individual transactions confidential with transparency for the regulator

**Endorsement** is the process in which a transaction is verified as “good”. Ensures that participants are happy to accept the transaction and prevents (e.g.) double-spending. In the real world, transactions are endorsed by a small number of participants (e.g. sender bank, receiver bank, payments provider) and must be completed in an appropriate timeframe.

![](https://379469498-files.gitbook.io/\~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FgfqoP1b6EjZ79OZ2MfkG%2Fuploads%2FKu5NXFfq4LxECFUZ4zKj%2Fpeers-diagram.png?alt=media\&token=854884d7-28f2-4e93-9a94-83091d1045c6)

### The architecture of a Fabric’s solution <a href="#id-68f3" id="id-68f3"></a>

To achieve consensus, and given that there is an assumption of partial trust in a Hyperledger Fabric network, Fabric uses a permissioned voting-based scheme, which achieves low-latency.

> _The endorsement policy defines the voting-based scheme to be used by peers and, consequently, the weight of each peer regarding the validity of a transaction._

The transaction flow, which follows the execute-order-validate paradigm, is as it follows:

1.  1\.

    **Transaction proposal**. A blockchain client, which represents an organization, creates a transaction proposal, and sends it to endorsement peers, as defined in the endorsement policy. The proposal contains information regarding the identity of the proposer, the transaction payload, a nonce, and a transaction identifier.
2.  2\.

    **Execute (endorsement)**: the endorsement consists of the simulation of the transaction. The endorsers produce a write-set, containing the keys and their modified values, and a read-set. The endorsement peers also check the correctness of transaction execution. The endorsement is sent as the proposal response and contains the write-set, read-set, the transaction ID, endorser’s ID, and the endorser’s signature. When the client collects enough endorsements (which need to have the same execution result), it creates the transaction and sends it to the ordering service. The endorsement phase eliminates any eventual non-determinism.
3.  3\.

    **Order**: after the endorsement, there is the ordering phase, performed by orderers. The ordering service checks if the blockchain client that submitted the transaction proposal has appropriate permissions (broadcast and receiving permissions), on a given channel. Ordering produces blocks containing endorsed transactions, in an ordered sequence, per channel. The ordering allows the network to achieve consensus. The orderer broadcasts the transaction’s outputs to all the peers.
4.  4\.

    **Validate**. Firstly, each peer validates the received transactions by checking if a transaction follows the correspondent endorsement policy. After that, a read-write conflict check is run against all transactions in the block, sequentially. For each transaction, it compares the versions of the keys in the read-set with those currently on the ledger. It checks if the values are the same. In case they do not match, the peers discard the transaction. Finally, the ledger is updated, in which the ledger appends the created block to its head. The ledger appends the results of the validity checks, including the invalid transactions.

{% embed url="https://docs.google.com/presentation/d/e/2PACX-1vRxTkz7cjb-aDes5rehLEAZamaJKiMhDQ_y5ZB1qanZWjvqm-trwdou1BIQuR8bOEThxYO1g_Q7BzBk/pub?delayms=3000&loop=false&start=true" %}

