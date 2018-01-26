# Setup

* macOS

  * Install Elixir - https://elixir-lang.org/install.html

```
brew install elixir;
elixir --version
```

* Tendermint
  * Install Tendermint v0.15.0 - https://github.com/tendermint/tendermint/wiki/Installation
    * Click "Install from Source" https://tendermint.com/downloads
    * Redirects to Install Tendermint https://tendermint.readthedocs.io/en/master/install.html
    * Install Tendermint From Source https://tendermint.readthedocs.io/en/master/install.html#from-source
    * Install GoLang @ https://golang.org/doc/install
    * Add GOPATH https://github.com/golang/go/wiki/SettingGOPATH
    * Add GoLang to $PATH
    * Note: Attempted to use Docker by encountered `shopt` error, as shown in Dockerfile

  ```
  go get --help
  go get -u -v github.com/tendermint/tendermint/cmd/tendermint
  ```

  * Run Tendermint

    * Initialise

```bash
tendermint init
```

  * View Tendermint Directory Root 

```bash
$ ls  ~/.tendermint

config.toml
data
genesis.json
priv_validator.json
```  

  * View TOML Configuration File

```bash
$ cat ~/.tendermint/config.toml
# This is a TOML config file.
# For more information, see https://github.com/toml-lang/toml

proxy_app = "tcp://127.0.0.1:46658"
moniker = "<MY_NETWORK_NAME>.local"
fast_sync = true
db_backend = "leveldb"
log_level = "state:info,*:error"

[rpc]
laddr = "tcp://0.0.0.0:46657"

[p2p]
laddr = "tcp://0.0.0.0:46656"
seeds = ""
```

  * View New Private Key from initialising Tendermint

```bash
$ cat ~/.tendermint/priv_validator.json | python -m json.tool
{
    "address": "E472...",
    "last_height": 18,
    "last_round": 0,
    "last_signature": {
        "type": "ed25519",
        "data": "B755..."
    },
    "last_signbytes": "7B22...",
    "last_step": 3,
    "priv_key": {
        "type": "ed25519",
        "data": "D11C..."
    },
    "pub_key": {
        "type": "ed25519",
        "data": "8373..."
    }
}
```

  * View Genesis File containing Public Key 

```bash
$ cat ~/.tendermint/genesis.json | python -m json.tool
{
    "app_hash": "",
    "chain_id": "test-chain-gZoesi",
    "genesis_time": "0001-01-01T00:00:00Z",
    "validators": [
        {
            "pub_key": {
                "type": "ed25519",
                "data": "8373..."
            },
            "power": 10,
            "name": ""
        }
    ]
}
```

  * Start Tendermint Single-Node Blockchain and Compile in-progress ABCI Application written in GoLang (i.e. Dummy App https://github.com/tendermint/abci)

```bash
$ tendermint node --proxy_app=dummy

I[01-22|21:35:03.283] Executed block                               module=state height=1 validTxs=0 invalidTxs=0
I[01-22|21:35:03.283] Committed state                              module=state height=1 txs=0 appHash=
I[01-22|21:35:04.295] Executed block                               module=state height=2 validTxs=0 invalidTxs=0
I[01-22|21:35:04.295] Committed state                              module=state height=2 txs=0 appHash=
```

  * Start Tendermint Single-Node Blockchain and Connect to and Compile a Non-GoLang ABCI App in-progress
    * Run App in another Socket Process

    * Start Tendermint Single-Node Blockchain and use Proxy Flag to specify Address of Socket the ABCI App is listening on

```bash
$ tendermint node --proxy_app=/var/run/abci.sock
```

  * Install ABCI-CLI with Go (includes example Dummy and Counter Apps) - https://tendermint.readthedocs.io/en/master/getting-started.html#dummy-a-first-example
   
    * Terminal 1

      ```bash
      go get -u github.com/tendermint/abci/cmd/abci-cli;
      abci-cli --help
      ```

    * Terminal 2

      ```bash
      tendermint init;
      tendermint unsafe_reset_all;
      tendermint node
      ```
    
    * Terminal 1

      ```bash
      curl -s localhost:46657/status
      ```
    


  * Deployment
    * TODO 
      * Kubernetes - https://github.com/tendermint/tools/tree/master/mintnet-kubernetes

# Tendermint App

* Run Tendermint Core (blockchain engine) & Custom Application - https://tendermint.readthedocs.io/en/master/getting-started.html#install





# About

* Tendermint

  * Diagrams
    * Tendermint in a (technical) nutshell - https://github.com/devcorn/hackatom/blob/master/tminfo.pdf

  * TendermintCore (BFT "consensus engine" Protocol that is mostly Asynchronous that uses a Simple State Machine https://tendermint.readthedocs.io/en/master/introduction.html#consensus-overview, and that communicates with an Application via TSP socket protocol that satisfies the ABCI)
    * About
      * Tendermint Core
        * Definition - a general purpose blockchain data structure (transactions batched in blocks where each block contains a cryptographic hash of the previous block, and the blocks form a chain) to replicate your Application on distributed systems (many machines) to provide fault tolerance
          * Secure Replication - Byzantine Fault Tolerant (BFT) since it still works when up to 1/3 of machines fail (from causes including hacking and malicious attacks)
          * Consistent Replication - all non-faulty machines may see same Transaction Log and computes same State
          * Consensus Engine - sharing Blocks and Transactions between Nodes to ensure recorded on all machines in same immutable order (blockchain)
          * Hosts Arbitrary States - may be used as plug-and-play replacement for consensus engines of other blockchain software (i.e. run any programming language implementation of Ethereum as an ABCI application using Tendermint consensus, i.e. Ethermint, Cosmos network)

    * Byzantine Fault Tolerant (BFT) Consensus engine
      * Note: Non Proof-of-Work (PoW) system
      * Reference: https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm
        * Summary
          * Block Data Structure - https://github.com/tendermint/tendermint/wiki/Block-Structure
            * `Header` - chain info and Last State
              * `ChainId` (string): name of the Blockchain, e.g. "tendermint"
              * `Height` (int): sequential Block Number starting with 1
              * `Time` (time): local time of the Proposer of this Block
              * `LastBlockHash` ([]byte): Block Hash of the Previous Block at `Height - 1`
              * `LastBlockPartsetHeader` (PartSetHeader): Partset Header of the Previous Block
              * `StateHash` ([]byte): State Hash of the State after processing this Block
                * `StateHash` is a hash derived from a encoding the State's Fields (i.e. `BondedValidators`, `UnbondingValidators`, `Accounts`, `ValidatorInfos`, and `NameRegistry`), using a [Simple Tree (Binary Tree)](https://github.com/tendermint/tendermint/wiki/Merkle-Trees#simple-tree-with-dictionaries) to merkelise a dictionary list of KV Pair structs.
                * `StateHash` is recursively included in the block `Header` and indirectly into the `BlockHash`

            * `Data` - Transactions of the Block that are any sequence of Bytes comprising:
              * `Txs` ([]`Tx`): list of Transactions. 
              Note: [ABCI Applications](https://github.com/tendermint/abci) (previuosly known as TMSP) may Accept or Reject a Tx
            * `LastCommit` - Comprises `Precommit` Signatures of the Previous Block 
              * `Precommits` ([]`Vote`): confirms that Consensus Decided for the Last Block was performed by over 2/3 (i.e. +2/3) of valid Voting Participants
                * `Vote`
                  * `Height` (int): Block Height being Decided on
                  * `Round` (int): Consensus Round number (starting with 0)
                  * `Type` (byte): Vote Type, either:
                    * `Prevote` or `Precommit` - [Vote Type](https://github.com/tendermint/tendermint/wiki/Block-Structure#vote) to Broadcast
                      i.e. `"type": 2`
                    * Note: Proof-of-Lock-Change (PoLC) is the Set of +2/3 `Prevote`'s for a Block, or `<nil>` at Node `(H, R)`
                  * `BlockHash` ([]byte): 
                    * Block Hash of a valid Block, or nil.
                    * Blockhash is a hash derived from a encoding the Block Header fields using a [Simple Tree (Binary Tree)](https://github.com/tendermint/tendermint/wiki/Merkle-Trees#simple-tree-with-dictionaries) to merkelise a dictionary list of KV Pair structs
                  * `BlockPartsetHeader` (PartSetHeader):
                    * PartSet Header, or x0000 if Block Hash is nil
                    * PartSet (inspired by LibSwift project) is used to propagate a Large File over a Gossip Protocol Network (i.e. multicast a message to all nodes in a network, where each node only sends the message to a few of the nodes)
                    * PartSet splits a Byteslice of Data (a Part) into a Transmission Parts List. Compute the Merke Root Hash of the Transmission Parts List. Verify that a Part is legitimately a Part of the Data by using the Merkle Root Hash of the Transmission Parts List prior to Forwarding the Part to other Peers (even before other Parts are known)  
                      i.e. `{ "hash": "XYZ", "total": 123 }`
                    * Example code: https://github.com/tendermint/tendermint/wiki/Block-Structure#commit
                  * `Signature` (Signature): `Signature` field of Vote's Transaction
                    * `sign-bytes`: JSON stringified (ordered) encoding (excluding the `Signature`) of Vote's Transaction
                      i.e. Example Precommit Vote:
                      `{ "chain_id": "tendermint","vote": { ... } }`

          * Consensus Process (State Machine) - https://tendermint.readthedocs.io/en/master/introduction.html#consensus-overview 
            * Deciding Next Block (at each Block Height `H`)
              * Rounds (Round-based Protocol) - each Round has a State Machine `RoundStep` representation (aka "Step") comprising Optimally 3x Steps + 2x Special Steps:
                * Round (sequence)
                  * Step 1: `Propose (H, R)` 
                    * Prerequisite, either:
                      * `CommitTime` + `timeoutCommit` (after `NewHeight` committed) OR
                      * `timeoutPrecommit`
                    * Start:
                      * Proposer (designated) proposes a Block at `(H, R)`
                    * End: 
                      * After `timeoutProposer` -> Step 2
                      * After `PoLC Round` receipt of Proposal Block and all Prevotes -> Step 2
                      * After "Common Exit Conditions"
                        * QUESTION - HOW? ALL THE COMMON EXIT CONDITIONS ARE POST- STEP 1
                          * https://matrix.to/#/!vIMgGaMqkLIWPCZvPF:matrix.org/$15167017365754249pwRYF:matrix.org
                  * Step 2: `Prevote (H, R)`
                    * Start:
                      * Validators broadcast their Prevote Vote
                    * Situations:
                      * Unlocking happens when Validators that have been Locked on a Block since `LastLockRound`
                      who also have a `PoLC` for something else at `PoLC-Round` where `LastLockRound < PoLC-Round < R`
                      * Prevote for a Block happens when a Validator is still Locked on a Block
                      * Prevote for a Proposed Block from `Propose (H, R)` happens when it is Valid
                      * Prevote of `<nil>` happens for a Proposed Block that is Invalid or received late
                    * End:
                      * After +2/3 Prevotes for a specific Block or `<nil>` -> Step 3
                      * After `timeoutPrevote` after receiving any +2/3 Prevotes -> Step 3
                      * After "Common Exit Conditions"
                  * Step 3: `Precommit (H, R)` after +2/3 Precommits for Block found
                    * Start:
                      * Validators broadcast their Precommit Vote
                    * Situation:
                      * Re-Locks / Changes Lock and Precommits Block `B` and Sets `LastLockRound = R` when Validator has PoLC `(H, R)` for specific Block `B`
                      * Unlocks and Precommits `<nil>` (i.e. obtained +2/3 Prevotes and waited but did not see PoLC for the Round) when Validator has PoLC at `(H, R)` for `<nil>`
                      * Lock remains and Precommits `<nil>` otherwise
                    * End:
                      * After +2/3 Precommits for `<nil>` -> Step 1 `Propose (H, R + 1)`
                      * After `timeoutPrecommit` after receiving any +2/3 Precommits -> Step 1 `Propose (H, R + 1)`
                      * After "Common Exit Conditions"
                * Special Step A: `Commit (H)` by:
                  * Setting `CommitTime = now`
                  * Waiting to receive Block and then Stage, Save, and Commit the Block -> Step B `NewHeight (H + 1)`
                * Special Step B: `NewHeight (H)`
                  * Move `Precommits` to `LastCommit`
                  * Increment Block Height
                  * Set `StartTime = CommitTime + timeoutCommit`
                  * Wait until `StartTime` to receive straggling commits -> Step 1 `Propose (H, 0)`
                    i.e. `NewHeight -> (Propose -> Prevote -> Precommit) + -> Commit -> NewHeight ->`
                * Note: See Diagram: https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm
              * Note: Multiple Rounds may be required to due to:
                * Problems
                  * Proposer (designated) was not online
                  * Proposer (designated) had proposed an Invalid Block
                  * Proposer (designated) had proposed a Valid Block that did not propogate in time
                  * Proposer (designated) had proposed a Valid Block but +2/3 of its Prevotes were not received in time (i.e. at least 1x Validator may have voted `<nil>` or performed a malicious votes) so there were Insufficient Validator Nodes when the Precommit step was reached
                  * Proposer (designated) had proposed a Valid Block and +2/3 of its Prevotes were received for sufficent nodes, but +2/3 of the Precommits for the Proposed Block were not received for sufficient Validator nodes.
                * Solutions
                  * Subsequent Round and Proposer (designated)
                  * Increasing Round parameters (i.e. Timeout) for successive rounds

          * Proposal 
            * Signed & Published by Proposer (designated) at each Round
            * Proposal at Node `(H, R)` comprises:
              * Block
              * Optional PoLC Round (which is less than `R`), included if Proposer is aware of one, and provides a hint to the network to allow "unlocking" of Nodes when safe to ensure "Proof of Liveness"

          * Proposer (NOT Validators)
            * Selected in proportion to their Voting Power (using round robin selection algorithm)
            * Reference: [Code](https://github.com/tendermint/tendermint/blob/develop/types/validator_set.go#L49)

          * Nodes - optionally connected in the network, are at a given:
            * Height `H` - Block Height being decided upon
            * Round `R` - Consensus Round number (starting from 0)
            * Step `S`
              * i.e. `(H, R, S)`
          
          * Peers - are Nodes that are directly connected to a Node

          * `Channel` (Throttled information)
            * `Connection` - between 2x Nodes

          * Epidemic Gossip Protocol - implemented by some Channels to update Peers on latest State of Consensus 
            * Nodes use PartSet algorithm (LibSwift inspired) to gossip/broadcast Parts (split Data) across the network for the current Round decision for a Proposed Block by a Proposer
            * Nodes gossip Prevote/Precommit Votes to enable Other Nodes to progress forward
              i.e. Node A (ahead of Node B) may send Node B Prevotes/Precommits for Node B's current/future Round
            * Nodes gossip Prevotes for Proposed Proof-of-Lock-Change (PoLC) Round (i.e. requires Set of +2/3 `Prevote`'s for a Block, or `<nil>` at Node `(H, R)`
            * Nodes gossip Block Commits for Older Blocks to Other Nodes that are lagging with a lower Blockchain Height
            * Nodes gossip `HasVote` messages opportunistically to hint to Peers what Votes they have
            * Nodes broadcast to all Peers their current State

          * Gossip Protocol Participants are "Validators"
            * Validators
              * Validators take turns as Proposer (delegate) Blocks containing Transactions and Voting on Blocks to be Committed on-chain at Block Height
              * Validators Vote to move to Next Round after waiting a Timeout interval (the only Synchronous aspect of the Protocol) to receive a complete Proposal Block from the Proposer 
              * Validators only progress after hearing from 2/3+ of the Validator Set

            * Blocks
              * Failure of a Block Commit causes the Protocol to move to the Next Round with a New Validator as Proposer (delegate) to propose the Next Block Height
              * Tendermint uses same mechanism for Block Commits as it does for Skipping to Next Round
              * Tendermint guarantees Safety without Validators ever committing a Block Commit that conflicts at the same Block Height on the assumption that 1/3- of Validators are BFT by using "Locking" rules that modulate the paths that may be followed in the Consensus Flow Diagram

            * Block Commit Failure
              * Causes
                * Proposer (delegate) may be Offline
                * Network may be slow
              * Solutions
                * Skip a Validator
            
            * Voting Stages required for Block Commit
              * Prevote 
                * Block Committed when 2/3+ of Validators Precommit for the same Block in the same Round.
                * "Polka" occurs when 2/3+ of Validators Prevote for the same Block
              * Precommit
                * Precommitted Blocks by a Validator are "Locked" on the Block and then must Prevote for the Block it is "Locked" on
                * Validators may only "Unlock" and Precommit for a New Block if there is a "Polka" for the New Block in a later Round
                * Precommits must be justified by a "Polka" in the same Round

            * Nodes
              * Non-Active Validators 
                * No Validator Private Key
                * Relays to Peers any Consensus Process information (i.e. meta-data, Proposals, Blocks, and Votes)
                * State (Current `H`, `R`, `S`)
                * Enable other Nodes to progress forward
              * Active Validator (aka "Validator-Node")
                * Validator Private Keys
                * Signs Votes
                * State (Current `H`, `R`, `S`)
                * Enable other Nodes to progress forward

            * Stake
              * Validator "Weight" varies in the Consensys Protocol of some systems
                * Voting Power Proportion may not be uniformly distributed across individual Validators 
              * Tendermint allows for creation of Proof-of-Stake systems through definition of a native Currency that denominates Voting Power. Cosmos Network is designed using this PoS mechanism across an array of cryptocurrencies implemented as ABCI Applications
                * Validators "bond" their Voting Power Currency holdings in a security deposit that may be destroyed if found to misbehave in the Consensys Protocol, which quantifies an economic security element whereby the Cost of Violating the assumption that 2/3- of Voting Power is Byzantine 

          * Proofs

            * Proof of Safety
              * Assuming -1/3 (less than 1/3) of Validators voting power are Byzantine
              * Validators may commit Block `B` at Round `R` after observing +2/3 (over 2/3) of Precommits at Round `R`
              * Unlikely for 1/3+ (over 1/3) of Honest Nodes Locked at Round `R' > R` to remain so until they observe PoLC at `R' > R`
              * -2/3 (less than 2/3) are available to Vote for anything other than Block `B`
              since 1/3+ (over 1/3) are Locked and Honest Nodes
              * CONFUSING?
            
            * Proof of Liveness
              * Assuming 1/3+ (over 1/3) of Validators are Locked on 2x different Blocks from different Rounds
              then a Proposer's `PoLC-Round` will Unlock Nodes that were Locked in previous Rounds
              * Proposer (designated) will become aware of a PoLC in a subsequent Round 
              * `timeoutProposalR` increments with Round `R` whilst Proposal size is capped, so network eventually is able to fully gossip the whole Proposal (both Block and PoLC)

            * TODO
              * Proof of Fork Accountability - https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm#proof-of-fork-accountability
              
              * Alternative Algorithm - https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm#alternative-algorithm
            
              * Censorship Attacks - https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm#censorship-attacks
              
              * Overcoming Forks and Censorship Attacks - https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm#overcoming-forks-and-censorship-attacks

    * Benefits:
      * Speed
        * Tendermint blocks can commit to finality in the order of 1 second
        * TendermintCore can handle transaction volume at the rate of 10,000 transactions per second for 250 byte transactions.
      * Security
        * Tendermint consensus is optimally BFT with accountability since liability may be determined when the blockchain forks
      * Scalability
        * Tendermint may be used for Sharding since it allows running parallel blockchains in parallel without the speed or security of each chain diminishing (unlike with PoW)  
        * Tendermint algorithm can scale to hundreds or thousands of validators (depending on desired block times), and improves over time with advances in bandwidth and CPU capacity.

  * Tendermint Generic Application BlockChain "Interface" (ABCI) is an API between Application Process and Consensus Process. ABCI has a Tendermint Socket Protocol (TSP aka Teaspoon) Implementation:
    * Run in separate Unix processes.
    * De-coupling (abstracting away) the Application State Details of a particular blockchain Application (using a socket protocol) from the Tendermint Core (consensus engine and P2P layers) to avoid a "monolithic" blockchain "stack" design so not limited to just using language of the blockchain stack that compile down to say the Turing-complete bytecode of the Ethereum Virtual Machine (EVM) (i.e. Serpent, Solidity)
    * Allows BFT replication of applications with Transactions processed in any programming language
    * Security guarantees
    * ABCI has 3x Primary Message Types (delivered from Tendermint Core to the ABCI, then ABCI replies with response messages)
      * DeliverTx Message
        * Transactions on the blockchain are delivered with it
        * Validation of Transactions received with the DeliverTx message must be validated by the Application against the current State, Application Protocol, and Transaction's Cryptographic Credentials
        * Valid Transactions update the Application State (i.e binding a value in a KV Store or updating the UTXO DB)
      * CheckTx Message
        * Only for Validating Transactions
        * Validates Transaction using Tendermint Core's Mempool
        * Only Relays Valid Transactions to Peers
        * Invalid Transactions may be caused if a Capabilities Based System is used but they have not renewed their capabilities with each Transaction, or CheckTx in an Application may return an error if a sequence number in a Transaction was not incremented
      * Commit Message
        * Computes the Cryptographic Commitment to the current Application State to be inserted into Next Block Header
        * Inconsistencies and errors in updating State appear as Blockchain Forks 
        * Secure Lightweight Client development is simplified since Merkle-Hash Proofs may be Verified against the Block Hash, which is Signed by a Quorum
    * Applications may have Multiple ABCI Socket Connections (3x created by Tendermint Core), including (see Diagram in https://tendermint.readthedocs.io/en/master/introduction.html#intro-to-abci):
      * Connection 1: Validates Transactions when broadcasting in the Mempool
      * Connection 2: Consensus Engine to run Block Proposals
      * Connection 3: Querying the Application State

    * Application Logic Deterministic
      * Transaction processing on blockchain must be Deterministic so Consensus is reached among Tendermint Core replica Nodes
      * Blockchain Developers should avoid Non-Determinism by using Linters and Static Analysers:
        * Avoid Random Number Generators without Deterministic Seeding
        * Avoid Race Conditions on threads
        * Avoid use of System Clock 
        * Avoid Uninitialised Memory (in Unsafe languages like C or C++)
        * Avoid Floating Point Arithmetic
        * Avoid Random Language Features 

  * Communication between TendermintCore and Tendermint Applications
    * TMSP (Simple Messaging Protocol)

# References

* Tendermint
  * TODO
    * TODO - TendermintCore Wiki - https://github.com/tendermint/tendermint/wiki/
    * TODO - Application Developers Guide - https://github.com/tendermint/tendermint/wiki/Application-Developers
    * TODO - Byzantine Consensus Algorithm - https://github.com/tendermint/tendermint/wiki/Byzantine-Consensus-Algorithm
    * TODO - Tendermint Documentation - https://tendermint.readthedocs.io/en/master/getting-started.html
    * TODO - Tendermint Documentation Release - https://media.readthedocs.org/pdf/tendermint/master/tendermint.pdf
    * TODO - Tendermine: BFT in the Age of Blockchains - https://allquantor.at/blockchainbib/pdf/buchman2016tendermint.pdf
    * TODO - Cosmos usage of Tendermint
      * https://cosmos.network/about/whitepaper
      * https://cosmos.network/about/faq
    * TODO - Merkle Tree Proof Calculation and Checking - https://github.com/yosriady/merkle_tree

* Scrap Notes:

```
cd $GOPATH/src/github.com/tendermint/tendermint
git tag -l
git checkout v0.14.0
```