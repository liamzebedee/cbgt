# cbgt

In order to make ultralight clients possible, we propose distributing blockchain (1) history and (2) state via a [GFS](https://liamzebedee.com/distsys/notes/google-file-system/)-like database design adapted for a Byzantine context.

The key design choices:
- the network is split into a master and a set of workers. The problem of storing data is split into fixed-size units of load balancing termed chunks, each chunk is replicated using Reed-Solomun encoding to workers, and the master operates as a control plane, maintaining indexing into the worker set.
- blockchain history is written regularly from blockchain nodes. Workers gate writes by ZK proofs; each write (upload of a block) is accompanied by a ZK proof which shows its validity.
- blockchain state is likewise accompanied by merkle proofs, which show the state leaf's update into the state root. State is stored according to an [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) approach, whereby each leaf is marked by its "time" (the block-height) marking its ordering across time. As this varies depending on the finality properties of each blockchain, this is application-specific. 
- storage in a byzantine context requires checks for data availability. we follow the classic approach of random sampling with a configurable checking period.
- the system scales storage and handles node failure by maintaining a central index of chunks stored in the master plane, and replicating them dynamically using Reed-Solomun encoding. node failure is detected by DA checks, as well as a requirement for regular heartbeats to a master. where there is worker failure, the master simply updates the chunk availability factor and polls other workers with free capacity to rebalance storage.
- the result is a system which has a scalable storage capacity, number of nodes, dynamic availability factor denoting the redundancy of data. Anyone can join as a node to store data, anyone can write data, but the writes can only be blocks or state leaves of a blockchain - as authorised by ZK proofs. 
- users can query this network to download the blockchain history with linear complexity by simply querying the master node for workers.
- users can query this network for state leaves without storing the entire state themselves, allowing them to simply download consensus proofs in order to track the correct chain. 
- with respect to economics, this is yet unsolved but we believe tractable. "blockspace" in this system (encompassing storage, bandwidth, redundancy) could be priced in a similar manner to lending markets, with a price curve that gets more expensive the closer the system is to 100% capacity (assuming a fixed availability factor of say 3.0).

This spec could feasibly increase access to Ethereum history and state without placing demand on full nodes.

With respect to tokenomics, there are a few options which suit different criteria for achieving a product-market fit and a sustainable network:
- Native token. The network has its own token, which it rewards node operators with, and uses for securing a master network of independent validators. This suits a product which is chain-agnostic.  
- Restaking. Using Eigenlayer AVS, we rent the security of the Ethereum validator set to secure the master network. Users pay in ETH for storage and access. This suits an Ethereum-centric product, as (1) it provides similar economic security level that can be easily measured in ETH and (2) it enables the product to be Ethereum aligned.
- Dual token restaking. Combining the benefits of both.


## Background reading.

- The need for ultralight clients, state expiry, the Ethereum portal network.
- Google File System v1, master/chunkserver delination, fixed-size units of load balancing, master control plane.
- GFS v2/S3, which both use Reed-Solomun encoding which allows for finer-grained replication factors 
- ZK proofs, Merkle proofs of Ethereum state leaves.
- Data availability approaches, namely Celestia/LazyLedger, Arweave's random sampling.
- [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) with respect to storing state leaves.
- Multidimensional resource pricing (re: economics), Aave's lending market pricing curve (expensive near 80% utilization)
