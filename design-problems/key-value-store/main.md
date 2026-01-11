# Key Value Store: Table of Contents

- [Key Value Store](#key-value-store)
  - [Step 1: Requirements and Design Scope](#step-1-requirements-and-design-scope)
  - [Step 2: Proposing High Level Design](#step-2-proposing-high-level-design)
    - [Designing a Single server Key-Value Store](#designing-a-single-server-key-value-store)
    - [Designing a Distributed Key-Value Store](#designing-a-distributed-key-value-store)
      - [CAP Theorem](#cap-theorem)
        - [Choosing between CP and AP](#choosing-between-cp-and-ap)
      - [System Components and Techniques](#system-components-and-techniques)
        - [1. Data Partitioning](#1-data-partitioning)
        - [2. Data Replication](#2-data-replication)
        - [3. Consistency Model](#3-consistency-model)
        - [4. Inconsistency Resolution](#4-inconsistency-resolution)
        - [5. Handling Failures](#5-handling-failures)
          - [A. Failure Detection](#a-failure-detection)
          - [B. Handling Temporary Failures](#b-handling-temporary-failures)
          - [C. Handling Permanent Failures](#c-handling-permanent-failures)
          - [D. Handling data center outages](#d-handling-data-center-outages)
  - [Step 3: Detailed Design](#step-3-detailed-design)
    - [Architecture Diagram](#architecture-diagram)
    - [Write Path](#write-path)
    - [Read Path](#read-path)
  - [Summary](#summary)




# Key Value Store

- A **key-value store** is a type of _non-relational_ database that uses a simple key-value pair to store data. 
- Each key is unique and is used to retrieve the corresponding value. 
- Keys can be plaint text or hashed values, while values can be any type of data, such as strings, numbers, JSON objects, or binary data.
- Key-value stores are designed for high performance, scalability, and flexibility, making them suitable for various applications such as caching, session management, and real-time analytics.

## Step 1: Requirements and Design Scope

- Support basic operations: `put(key, value)` and  `get(key)`.
- The size of key-value pair is small (few KBs).
- Ability to store big data (few TBs).
- High availability: System responds quickly even during failures.
- High Scalability: System can be scaled to support large data set.
- Automatic Scaling: System should automatically scale up/down based on load.
- Tunable Consistency: Ability to choose between strong and eventual consistency based on use case.
- Low Latency: Fast read and write operations.

## Step 2: Proposing High Level Design
### Designing a Single server Key-Value Store

- Store key-value pairs in hash table, which keeps everything in memory.
- Memory access is fast, so read and write operations are quick.
- But memory is expensive and limited, so this design is not suitable for large data sets.
- Optimizations?
  - Data Compression: Compress data before storing to save memory.
  - Store only frequently accessed data in memory, and move less frequently accessed data to disk.
- Even with these optimizations, a single server key-value store is not suitable for large data sets.

### Designing a Distributed Key-Value Store

- To handle large data sets, we need to distribute data across multiple servers.
- We need to understand CAP theorem before designing a distributed key-value store.

#### CAP Theorem

Refer [here](./../../Readme.md#cap-theorem-brewers-theorem)

### System Components and Techniques

#### 1. Data Partitioning
- For large applications, data is partitioned across multiple servers to distribute load and improve performance.
- Challenges with partitioning:
  - How to distribute data across multiple servers evenly?
  - How to minimize data movement when nodes are added or removed?
- Techniques for data partitioning:
  - **Range-based Partitioning**: Data is partitioned based on key ranges. For example, keys starting with 'A' to 'M' are stored in server 1, and keys starting with 'N' to 'Z' are stored in server 2.
    - Pros: Simple to implement and understand.
    - Cons: Can lead to hot spots if certain key ranges are accessed more frequently.
  - **Hash-based Modulo Partitioning**: A hash function is applied to the key to determine which server will store the data. For example, `hash(key) % number_of_servers` can be used to determine the server.
    - Pros: Distributes data evenly across servers.
    - Cons: When a server is added or removed, a large portion of data needs to be moved.
  - **Consistent Hashing**: A more advanced technique that minimizes data movement when servers are added or removed. Servers and keys are mapped to a circular hash space, and each key is assigned to the next server in a clockwise direction.
    - Pros: 
      - Automatic Scaling: Minimizes data movement during scaling operations.
      - Heterogeneity: the number of virtual nodes for a server is proportional to its capacity. For example, a server with double the capacity can have double the virtual nodes, allowing it to handle more keys.
    - Cons: More complex to implement.
- For further reading refer [Consistent Hashing](./../consistent-hashing/main.md).


#### 2. Data Replication
- To ensure high availability and fault tolerance, data is replicated across multiple servers.
- In consistent hashing, each key can be replicated to the next `N` (configurable) servers in a clockwise direction on the hash ring.
- This way, if one server goes down, the data can still be accessed from another server
- With virtual nodes, the first `N` distinct servers in a clockwise direction are chosen for replication.
- Nodes in same data center often fail at same time due to power/network issues. To handle this, replicas can be placed in different data centers and data centers are connected through high speed networks.
- For further reading refer [Data replication and Migration in Distributed Systems](./../../concepts/data-replication-and-migration.md).

#### 3. Consistency Model
- In a distributed key-value store, we can choose between strong consistency and eventual consistency based on the use case.
- **Strong Consistency**: Ensures that all clients see the most recent write. This can be achieved using techniques like quorum-based replication, where a write is considered successful only when a majority of replicas acknowledge the write.
- **Eventual Consistency**: Ensures that all replicas will eventually converge to the same value, but there may be a delay before all replicas are updated. This can be achieved using techniques like asynchronous replication, where writes are propagated to replicas in the background.
- The choice between strong and eventual consistency depends on the specific requirements of the application.
- Consider
  ```
    N -> The number of replicas
    W -> The number of replicas that must acknowledge a write for it to be considered successful
    R -> The number of replicas that must be read to ensure the most recent write is read
  ```
- To achieve strong consistency, we can set `W + R > N`. This ensures that there is at least one replica that has acknowledged the write and is read during a read operation.
- To achieve eventual consistency, we can set `W + R <= N`. This allows for faster writes and reads, but there may be a delay before all replicas are updated.
- if R = 1 and W = N, The system is optimized for fast read.
- if W = 1 and R = N, The system is optimized for fast write.
- For further reading refer [Consistency Models in Distributed Systems](./../../concepts/consistency.md).
    

- Out of 3 consistency models (Strong, Eventual, Causal), key-value stores typically use Eventual consistency models.
- From concurrent writes, eventual consistency allows inconsistent values to enter the system and forces client to read the values to reconcile them.

#### 4. Inconsistency Resolution

- In eventually consistent systems, replicas may hold conflicting versions of the same data due to:
  - Concurrent writes 
  - Network partitions 
  - Delayed replication 
- To resolve, systems track versions of each object instead of overwriting blindly.

- Techniques for versioning:
  
  | Technique                                       | Description                                               | Working                                                                          | Pros                                                           | Cons                                                                                                        | Real-world Analogy                                                                          |
  |-------------------------------------------------|-----------------------------------------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
  | **Simple Version Numbers**                      | Each update increments a version counter.                 | Replicas compare counters; higher counter overwrites lower.                      | Simple to implement.                                           | Cannot capture concurrent writes properly (overwrites valid updates).                                       | Book editions: higher edition replaces the previous one, but parallel drafts are lost.      |
  | **Timestamps (LWW)**                            | Each version is tagged with a timestamp.                  | Latest timestamp chosen as the correct value.                                    | Easy and efficient.                                            | Requires synchronized clocks; may lose valid updates.                                                       | “Last modified time” in a file system – the newest timestamp wins.                          |
  | **Vector Clocks**                               | Each version tagged with a list of (node, counter) pairs. | Tracks causal history; detects whether versions are concurrent or one dominates. | Can detect concurrent updates and preserve causality.          | Complex to implement; more storage overhead.(set threshold for length, discard old pairs if limit exceeded) | Google Docs edit history: can detect parallel edits and keep both until merged.             |
  | **Conflict-free Replicated Data Types (CRDTs)** | Special data structures designed for automatic merging.   | CRDTs define merge rules ensuring eventual consistency without conflicts.        | Automatically resolves conflicts; strong eventual consistency. | Limited to specific data types (counters, sets, maps).                                                      | Shared shopping cart: items from different users are merged automatically without conflict. |

---

---
#### 5. Handling Failures
- In a distributed key-value store, we need to handle various types of failures to ensure high availability and fault tolerance.

##### A. Failure Detection
1. **All-to-All heartbeat Mechanism**: 
   - It requires atleast two independent failure detectors to avoid false positives. 
   - An all-to-all multicast heartbeat mechanism can be used where each node sends heartbeat messages to all other nodes at regular intervals. 
   - If a node does not receive a heartbeat from another node within a certain timeout period, it can mark that node as failed.
   ![heartbeat mechanism](../../images/keyValueStore/keyValue-1.png)

2. **Gossip Protocol**:
  - Each node maintains a node membership list, which contains member IDs and heartbeat
    counters. 
  - Each node periodically increments its heartbeat counter.
  - Each node periodically sends heartbeats to a set of random nodes, which in turn
    propagate to another set of nodes.
  - Once nodes receive heartbeats, membership list is updated to the latest info.
  - If the heartbeat has not increased for more than predefined periods, the member is
    considered as offline.
  - For example
    - Node s0 maintains a node membership list shown on the left side.
    - Node s0 notices that node s2’s (member ID = 2) heartbeat counter has not increased for a
      long time.
    - Node s0 sends heartbeats that include s2’s info to a set of random nodes. Once other
      nodes confirm that s2’s heartbeat counter has not been updated for a long time, node s2 is
      marked down, and this information is propagated to other nodes.
      ![gossip protocol](../../images/keyValueStore/keyValue-2.png)

##### B. Handling Temporary Failures
- After failure detection, we need to handle temporary failures to ensure high availability.
- In strict quorum approach, read and write operations are blocked if the required number of replicas are not available.
- In "sloppy quorum" approach, read and write operations can proceed as long as a subset of replicas are available. (system chooses first W healthy servers for writes and first R healthy servers for reads)
- Techniques:
  - **Hinted Handoff**: When a node is temporarily unavailable, writes intended for that node are stored on a different node (the "hint"). Once the original node comes back online, the hints are forwarded to it.
  - **Read Repair**: During read operations, if a node detects that it has stale data compared to other replicas, it can update its data to the latest version.


##### C. Handling Permanent Failures
- In case of permanent failures, we need to ensure that data is not lost and the system remains available.
- Techniques:
- **Anti-Entropy**: 
  - A background process that periodically compares data between replicas and synchronizes any differences.
  - A merkle tree can be used to efficiently identify differences between replicas.
  > A `hash tree` or `Merkle tree` is a tree in which every non-leaf
  node is labeled with the hash of the labels or values (in case of leaves) of its child nodes.
  Hash trees allow efficient and secure verification of the contents of large data structures

- Anti-entropy ensures that live replicas periodically compare data:
  - Full dataset hashes (Merkle trees), or
  - Version metadata (vector clocks)
- This comparison allows replicas to:
  - Detect which updates are missing
  - Identify divergent keys or ranges

- Use of Merkle Tree:
  - Replica comparison:
    - If two replicas have same root hash, data is identical. 
    - If root hashes differ, check children recursively until differences are found. 
    - Only mismatched branches need syncing.

  - Proof of data integrity:
    - A Merkle proof allows you to verify a single block belongs to the dataset without downloading all data.

##### D. Handling data center outages
- To handle data center outages, replicas can be placed in different data centers.
- Data centers are connected through high speed networks to ensure low latency access to replicas.
- In case of a data center outage, clients can still access replicas in other data centers.


## Step 3: Detailed Design

### Architecture Diagram
- Clients communicate with the key-value store through simple APIs: get(key) and put(key,
value).
- A coordinator is a node that acts as a proxy between the client and the key-value store.
- Nodes are distributed on a ring using consistent hashing.
- The system is completely decentralized so adding and moving nodes can be automatic.
- Data is replicated at multiple nodes.
- There is no single point of failure as every node has the same set of responsibilities.
![architecture-diagram](../../images/keyValueStore/arch-diagram.png)

- Since the system is decentralized, any node can act as a coordinator.
- Each node has same set of responsibilities.
![Node-responsibilities](../../images/keyValueStore/node-responisibilities.png)

### Consistent Hashing Flow
- Hash the Key 
  - Compute a hash of the key → hash(key). 
  - Example: hash("user123") → 856. 
- Map Nodes to a Ring 
  - All nodes are placed on a circular hash ring using a hash function. 
  - Example: Node A → 100, Node B → 500, Node C → 900. 
- Locate the Key’s Node 
  - Move clockwise on the ring to find the first node ≥ key’s hash. 
  - If hash(key)=856 → closest node clockwise is Node C (900). 
- Replication (Optional)
  - Store copies of the key on the next N nodes clockwise for fault tolerance. 
- Node Joins/Leaves 
  - If a new node joins, only nearby keys need to be redistributed. 
  - Minimizes data movement compared to simple modulo hashing.

### Write Path

1. Client sends a `put(key, value)` request to any node (coordinator).
2. Coordinator uses consistent hashing to find the responsible node(s).
3. The write request is persisted on a commit log file.
   > Note: A commit log is an append-only file that records every change made to the database. It ensures durability by allowing recovery of data in case of a crash.
4. Data is saved in the memory cache.
5. When the memory cache is full or reaches a predefined threshold, data is flushed to
   SSTable on disk. 
   > Note: A [sorted-string table (SSTable)](https://www.igvita.com/2012/02/06/sstable-and-log-structured-storage-leveldb/) is a sorted list of <key, value>
      pairs. 

![write-path](../../images/keyValueStore/write-path.png)


### Read Path

1. After a read request is directed to specific node, it first checks its memory cache.
   ![img.png](../../images/keyValueStore/readPath1.png)
2. If the key is not found in memory, system checks the bloom filter.
3. The bloom filter is used to figure out which SSTable files might contain the key.
   > Note: A [bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) is a space-efficient probabilistic data structure used to test whether an element is a member of a set. It can yield false positives but not false negatives.
4. If the bloom filter indicates that the key might be in an SSTable, the system checks the SSTable index to find the exact location of the key.
5. Finally, the system retrieves the value from the SSTable on disk.

    ![img_1.png](../../images/keyValueStore/readPath2.png)

## Summary

| Goal/Problems               | Technique                                                |
|-----------------------------|----------------------------------------------------------|
| Ability to store big data   | Use consistent hashing to spread the load across servers |
| High availability reads     | Data replication, Multi-data center setup                |
| Highly available writes     | Versioning and conflict resolution with vector clocks    |
| Dataset partition           | Consistent Hashing                                       |
| Incremental scalability     | Consistent Hashing                                       |
| Heterogeneity               | Consistent Hashing                                       |
| Tunable consistency         | Quorum consensus                                         |
| Handling temporary failures | Sloppy quorum and hinted handoff                         |
| Handling permanent failures | Merkle tree                                              |
| Handling data center outage | Cross-data center replication                            |
